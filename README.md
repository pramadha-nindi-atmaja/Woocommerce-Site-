# Shopify SaaS Clone using WordPress, WooCommerce, WPCS & k8s

A proof-of-concept showing how to build a Shopify-style SaaS product on top of WordPress. Customers buy a subscription in a WooCommerce store, and a new WordPress website (a "tenant") is automatically provisioned for them through the [WPCS.io](https://wpcs.io) API. When the subscription expires, the tenant is removed.

## What's inside

```
.
├── docker-compose.yml        # Local WordPress 6.0 (PHP 8.0) + MySQL 5.7 stack
├── LICENCE
└── src/                      # The "WPCS WooCommerce Subscriptions" WordPress plugin
    ├── index.php             # Plugin entry point: defines WPCS credentials constants and boots the classes
    ├── composer.json         # Dependencies (wpcs/php-api-wrapper) and PSR-4 autoloading
    └── Core/
        ├── TenantsSubscriptionController.php
        ├── WPCSSettingsPage.php
        ├── VersionsService.php
        └── WooCommerceMetaBoxes.php
```

### Plugin components

| File | Responsibility |
| --- | --- |
| `index.php` | Plugin header and bootstrap. Loads the Composer autoloader, reads the WPCS region, API key and API secret from WordPress options, and instantiates the classes below. |
| `Core/TenantsSubscriptionController.php` | The core flow. Adds a required **Website Name** field to the WooCommerce checkout and saves it on the order. When a subscription is created (Subscriptions for WooCommerce hook), it creates a tenant via the WPCS API using the product's selected version, generates an admin password, and emails the customer their admin URL and credentials. When the subscription expires, it deletes the tenant. |
| `Core/WPCSSettingsPage.php` | Adds a **WPCS.io** page to the WordPress admin menu where you enter the WPCS region, API key and API secret. |
| `Core/VersionsService.php` | Fetches the list of available site versions (templates) from the WPCS API. |
| `Core/WooCommerceMetaBoxes.php` | Adds a **WPCS Version Selector** box to the WooCommerce product editor so each product can be linked to a WPCS version. Only versions with status `Done` are listed. |

### How it works

1. In the admin, set your WPCS credentials under **WPCS.io**.
2. On a WooCommerce product, choose which WPCS version to provision.
3. A customer checks out, entering a website name.
4. When the subscription is created, the plugin calls WPCS to create the tenant and emails the customer their login details.
5. When the subscription expires, the plugin deletes the tenant.

### Tech stack

- WordPress and WooCommerce
- Subscriptions for WooCommerce (`wps_sfw_*` hooks)
- [WPCS.io](https://wpcs.io) (Kubernetes-based WordPress hosting API) via `wpcs/php-api-wrapper`
- Docker / Docker Compose for local development
- Composer for PHP dependencies

## Local Development

### Required tools

- Docker and Docker-Compose
- Composer
- `fswatch` & `rsync`

### Steps

- `git clone` the project
- Run `composer install` inside the `src` directory
- Create `.db` & `dist` directories in the root
- Run `docker-compose up` (WordPress is served on http://localhost:3000)
- Run the following command to sync files between `src` and `dist`:

```
fswatch -o src | xargs -n1 -I{} rsync -a src/ dist/plugins/wpcs-woo-subscriptions
```

- Activate the plugin in the WordPress admin, then install WooCommerce and Subscriptions for WooCommerce.
