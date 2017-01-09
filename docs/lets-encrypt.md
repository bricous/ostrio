Let's Encrypt - Free SSL (TLS) certificates
======

In order to make World Wide Web safer and faster, we strongly recommend to use HTTPS protocol for your website, and add H2 protocol support to your web-server.

This tutorial will show how to use service by Let's Encrypt. Which let's you to generate fully qualified SSL (TLS) certificate for free.

*Examples is given for Debian/Ubuntu Linux an Nginx*

It is recommended to install certificates via [Certbot](https://certbot.eff.org) app.

#### Install Certbot:
```shell
# Debian 8
apt-get install certbot -t jessie-backports

# Debian 7
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
# For more info go to:
# https://certbot.eff.org/#debianwheezy-nginx
```

#### Set Nginx configuration:
```nginx
# /etc/nginx/sites-available/example.conf
server{
  listen 80;
  listen [::]:80;
  server_name example.com;
  root /var/www/example;
  location /.well-known/acme-challenge/ {
    try_files $uri =404;
  }
}
```

#### Make sure root exists:
```shell
mkdir -p /var/www/example
```

#### Generate certificate:
```shell
# --email admin@example.com <- Email for important notifications
# -w /var/www/example/ <- Nginx host "root"
# -d example.com <- Domain name
certbot-auto certonly --email ceo@veliovgroup.com --webroot -w /var/www/default/ -d 185.48.236.86

# This command will return
# path to directory with certificates
# like: /etc/letsencrypt/live/example.com/
```

#### Update Nginx configuration:
```nginx
# /etc/nginx/sites-available/example.conf
server{
  listen 80;
  listen [::]:80 ipv6only=on;
  server_name example.com;

  # Redirect all requests to HTTPS
  return 301 https://$http_host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name example.com;

  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

  ssl                 on;

  # Everything below can be moved to nginx.conf
  # To apply to all hosts with SSL

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:DES-CBC3-SHA:!RC4:!aNULL:!eNULL:!MD5:!EXPORT:!EXP:!LOW:!SEED:!CAMELLIA:!IDEA:!PSK:!SRP:!SSLv:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;
  ssl_session_cache   shared:SSL:32m;
  ssl_session_timeout 4h;
  ssl_buffer_size     1400;
  ssl_session_tickets on;
  ssl_dhparam         /etc/nginx/ssl/dhparam.pem;

  ssl_stapling        on;
  ssl_stapling_verify on;

  # ... Other settings of your web-app
}
```

#### Enable example.conf:
```shell
ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/example.conf
```

#### Enable http2 (H2):
```nginx
# create or edit /etc/nginx/sites-available/default
server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2 ipv6only=on;
  server_name _;

  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

  ssl                 on;

  return 444;
}
```

#### Generate `dhparam`:
```shell
# Go to /etc/nginx/ssl/
mkdir -p /etc/nginx/ssl/
cd /etc/nginx/ssl/

# Generate dhparam
# Note: it may take up to few hours
openssl dhparam -out dhparam.pem 4096
```

#### Set permissions:
Use `chown` to set files owner, usually `www-data` for Nginx
```shell
chown -R www-data:www-data /etc/nginx
chmod -R 644 /etc/nginx
find /etc/nginx -type d -exec chmod 700 {} \;
chmod -R 600 /etc/nginx/ssl

chown -R www-data:www-data /var/www/example
```

#### Test configuration:
```shell
service nginx configtest
nginx -t
```

#### Restart Nginx to apply changes:
```shell
service nginx restart
```

#### Test SSL (TLS) setup:
 - Go to [ssllabs.com](https://www.ssllabs.com/ssltest/index.html)
 - Enter your domain
 - You should get A+ rating with this setup