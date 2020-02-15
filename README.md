# playbook: install wordpress
Pre -requests files create on your local machine
* vhost.tmpl
* wp-config.php.tmpl
## cat vhost.tmpl
```
<virtualhost *:80>
  servername {{ domain }}
  documentroot /var/www/html/{{ domain }}
  directoryindex index.html index.php
</virtualhost>

```
