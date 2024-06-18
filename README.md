# nextcloud_install
Nextcloud installation

## Install on Debian 12 stable

### Update repository list and Upgrade distro

```
# apt update
# apt upgrade
```

### Install some prerequist packages.

```
# apt install unzip nginx mysql-server
```

### Install required php packages

```
# apt install php php-cli php-common php-json php-fpm php-curl php-mysql php-gd php-opcache php-xml php-zip php-mbstring
```

### Check php installation
```
php -v
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

### Database 

```
apt install madiadb-server
sudo systemctl start mysql
sudo systemctl enable mysql
```

```
sudo mariadb -u root
```

Inside mariadb shell write these commands to create database and user.

```
CREATE DATABASE nextcloud;
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'username'@'localhost';
FLUSH PRIVILEGES;
exit;
```

check nginx webserver, start and schedule to run on startup.

```
# nginx -v
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Nextcloud Steps

#### Download nextcloud archive and place on **/var/www** 

```
wget https://download.nextcloud.com/server/releases/latest.zip -P /var/www/
```

#### Set permission for webserver

```chown -R www-data:www-data /var/www/nextcloud```

#### Create nginx config

**/etc/nginx/sites-enabled/nextcloud**

```
upstream php-handler {
        server unix:/var/run/php/php-fpm.sock;
}

server {
        listen 80;
        listen [::]:80;
        server_name _;

        root /var/www/nextcloud;
        index index.php index.html /index.php$request_uri;

        # Limit Upload Size
        client_max_body_size 512M;
        fastcgi_buffers 64 4K;

        # Gzip Compression
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        # Recommended Security Headers
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;
        fastcgi_hide_header X-Powered-By;

        # Recommended Hidden Paths
        location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
        location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)              { return 404; }

        location ~ \.php(?:$|/) {
                fastcgi_split_path_info ^(.+?\.php)(\/.*|)$;
                set $path_info $fastcgi_path_info;
                try_files $fastcgi_script_name =404;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $path_info;
                fastcgi_param modHeadersAvailable true;
                fastcgi_param front_controller_active true;
                fastcgi_pass php-handler;
                fastcgi_intercept_errors on;
                fastcgi_request_buffering off;
                include fastcgi_params;
                proxy_connect_timeout 600s;
                proxy_send_timeout 600s;
                proxy_read_timeout 600s;
                fastcgi_send_timeout 600s;
                fastcgi_read_timeout 600s;
        }

        # Cache-Control on Assets
        location ~ \.(?:css|js|woff2?|svg|gif|map)$ {
                try_files $uri /index.php$request_uri;
                add_header Cache-Control "public, max-age=15778463";
                expires 6M;
        }

        location ~ \.(?:png|html|ttf|ico|jpg|jpeg|bcmap)$ {
                try_files $uri /index.php$request_uri;
        }

        location / {
                try_files $uri $uri/ /index.php$request_uri;
        }
}
```

#### enable nginx config and restart webserver

```
ln -s /etc/nginx/sites-available/nextcloud -t /etc/nginx/sites-enabled/
systemctl restart nginx
```

## NextCloud Setup
Open this address on browser 

http://${yourip}

for example, http:192.168.5.30

Set new user and password
for example
```
user: admin
password: admin
```
Input database fields 

```
databasename: nextcloud
username: username
password: password
```

And then click on Install recommended appas.

Now nextcloud is starts.


---

# Dockerize

https://github.com/nextcloud/all-in-one#how-to-use-this

