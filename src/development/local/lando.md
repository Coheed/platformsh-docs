# Using Lando for local development

[Lando](https://github.com/lando/lando) is a container-based local development toolchain that plays nicely with Platform.sh.  It is maintained by [Tandem](https://thinktandem.io), a 3rd party agency, but is a viable option for most Platform.sh projects.

See the [Lando documentation](https://docs.devwithlando.io/) for installing and setting up Lando on your system.

Lando will ask you to create a `.lando.yml` file in your application root, which functions similarly to the `.platform.app.yaml` file.  (Note the different file extension.)  It is safe to check this file into your Git repository as Platform.sh will simply ignore it.

If your application is one of those with a specific "recipe" available from Lando, you can use that directly in your `.lando.yml` file.  It can be customized further as needed for your application, and some customizations are specific to certain applications.

## `.lando.yml` configuration

In particular, we recommend:

```yaml
# Name the application the same as in your .platform.app.yaml.
name: app
# Use the recipe appropriate for your application.
recipe: drupal8

config:
  # Lando defaults to Apache. Switch to nginx to match Platform.sh.
  via: nginx

  # Set the webroot to match your .platform.app.yaml.
  webroot: web

  # Lando defaults to the latest MySQL release, but Platform.sh uses MariaDB.
  # Specify the version to match what's in services.yaml.
  database: mariadb:10.1
```

## Downloading data from Platform.sh into Lando

In most cases downloading data from Platform.sh and loading it into Lando is straightforward.  If you have a single MySQL database then the following two commands, run from your application root, will download a compressed database backup and load it into the local Lando database container.

```bash
platform db:dump --gzip -f database.sql.gz
lando db-import database.sql.gz
```

Rsync can download user files easily and efficiently.  See the [exporting tutorial](/tutorials/exporting.md) for information on how to use rsync.

Then you need to update your `sites/default/settings.local.php` to configure your codebase to connect to the local database that you just imported:

```php
/* Working in local with Lando */
if (getenv('LANDO') === 'ON') {
  $lando_info = json_decode(getenv('LANDO_INFO'), TRUE);
  $settings['trusted_host_patterns'] = ['.*'];
  $settings['hash_salt'] = 'CHANGE THIS TO SOME RANDOMLY GENERATED STRING';
  $databases['default']['default'] = [
    'driver' => 'mysql',
    'database' => $lando_info['database']['creds']['database'],
    'username' => $lando_info['database']['creds']['user'],
    'password' => $lando_info['database']['creds']['password'],
    'host' => $lando_info['database']['internal_connection']['host'],
    'port' => $lando_info['database']['internal_connection']['port'],
  ];
}
```
