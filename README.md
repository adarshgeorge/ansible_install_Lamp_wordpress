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
### # vim vhostcreation.yml
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

## Second playbook: Install MySQL
### # vim mysql.yml
```
---
  - name: "Installing MySQL to Remote Host"
    hosts: all
    become: yes
    vars:
      - mysql_pswd: mysql1232
        db_user: wp-user
        db_name: wp-db
        db_password: wp123h
        domain: adamz.com
    tasks:
      - name: "installing MySQL"
        yum:
          name:
          - mariadb-server
          - MySQL-python


          state: present
      - name: "starting"
        service:
          name: mariadb
          state: started
          enabled: yes
      - name: Resetting mysql password"
        ignore_errors: true
        mysql_user:
          login_user: "root"
          login_password: ""
          user: "root"
          password: "{{ mysql_pswd }}"

      - name: "Removing anonymous user"
        mysql_user:
          login_user: root
          login_password: "{{ mysql_pswd }}"
          user: ""
          state: absent
          host_all: true

      - name: "Creating database"
        mysql_db:
          login_user: "root"
          login_password: "{{ mysql_pswd }}"
          name: "{{ db_name }}"
          state: present

      - name: "Adding wordpress User to MySQL"
        mysql_user:
          login_user: "root"
          login_password: "{{ mysql_pswd }}"
          name: "{{ db_user }}"
          password: "{{ db_password }}"
          host: "localhost"
          priv: "{{ db_name}}.*:All "
```

## Third : Wordpress download and configure

### # vim wp-install.yml

```
---
  - name: "Install Wordpress"
    hosts: all
    become: yes
    vars:
      - db_user: wp-user
        db_name: wp-db
        db_password: wp123h
        domain: adamz.com
    tasks:
      - name: "Downloading Wordpress Tar file"
        get_url:
          url: https://wordpress.org/wordpress-4.7.8.tar.gz
          dest: /tmp/wordpress.tar.gz
          remote_src: true
      - name: "Extracting tar file"
        unarchive:
          src: /tmp/wordpress.tar.gz
          dest: /tmp/
          remote_src: yes
      - name: "Copying to DocRoot"
        copy:
          src: /tmp/wordpress/
          dest: "/var/www/html/{{ domain }}"
          remote_src: true
          owner: apache
          group: apache




      - name: "Genaratigng wp-config"
        template:
          src: wp-config.php.tmpl
          dest: "/var/www/html/{{ domain }}/wp-config.php"
          owner: apache
          group: apache
      - name: "Post installation clear"
        file:
          path: "{{ item }}"
          state: absent
        with_items:
          - /tmp/wordpress.tar.gz
          - /tmp/wordpress
```

## Execute the playbook one by one

```
# ansible-playbook -i hosts vhostcreation.yml
```
