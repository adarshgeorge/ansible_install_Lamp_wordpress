# playbook: install wordpress
Pre -requests files create on your local machine
* vhost.tmpl
* wp-config.php.tmpl
### # cat vhost.tmpl
```
<virtualhost *:80>
  servername {{ domain }}
  documentroot /var/www/html/{{ domain }}
  directoryindex index.html index.php
</virtualhost>

```
### # vim  wp-config.php.tmpl

```
<?php
define( 'DB_NAME', '{{ db_name }}' );

/** MySQL database username */
define( 'DB_USER', '{{ db_user }}' );

/** MySQL database password */
define( 'DB_PASSWORD', '{{ db_password }}' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
$table_prefix = 'wp_';

define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

## First playbook:  Install httpd and add Virtual Host

```
Vim vhostcreation.yml
---
  - name: Install httpd and add Virtual Host"
    hosts: all
    become: yes
    vars:
      domain: adamz.com
    tasks:
      - name: "Install Apache"
        yum:
          name:
            - httpd
            - php
            - php-mysql
          state: present
      - name: "starting service and enabled"
        service:
          name: httpd
          state: restarted
          enabled: yes
      - name: "Adding Vhost"
        template:
          src: vhost.tmpl
          dest: "/etc/httpd/conf.d/{{ domain }}.conf"
          owner: apache
          group: apache
      - name: "Display page"
        file:
          path: "/var/www/html/{{ domain }}"
          state: directory
          owner: apache
          group: apache
      - name: "restarting httpd service"
        service:
          name: httpd
          state: restarted
```


