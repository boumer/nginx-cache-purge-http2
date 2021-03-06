# nginx-cache-purge-http2

This is a base Docker Nginx (v1.11.9) image that supports both Cache Purging and HTTP/2 connections.

It is Official Docker Nginx compiled with [FRiCKLE's ngx_cache_purge module](https://github.com/FRiCKLE/ngx_cache_purge), [Nginx's http_v2_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html) and the [OpenSSL v1.0.2k library](https://www.openssl.org/) (providing HTTP/2 support via ALPN) and supports both Cache Purging and HTTP/2 connections.

---

##How to use this image:

Use this image as a Dockerfile base (i.e.,```FROM stcox/nginx-cache-purge-http2```) for building a final image that's been configured to:

1. Purge content from FastCGI, proxy, SCGI and uWSGI caches, and/or
2. Enable HTTP/2 via OpenSSL ALPN.

---

Bake your Nginx configuration files and SSL certificates directly into your image using ```COPY``` in a Dockerfile, or use ```sed``` commands within ```docker-entrypoint.sh``` to configure Nginx on container startup.

**Dockerfile Example:**
```
# Dockerfile Example

FROM stcox/nginx-cache-purge-http2 # base image

#conf
COPY default.conf         /etc/nginx/conf.d/default.conf
COPY docker-entrypoint.sh /entrypoint.sh

#ssl
COPY example.com.crt /etc/nginx/ssl/example.com.crt
COPY example.com.key /etc/nginx/ssl/example.com.key

ENTRYPOINT ["/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

---

**Enable Cache Purging:**

1. Configure caching (FastCGI, proxy, SCGI, or uWSGI).
  - e.g. - WordPress Multisite FastCGI Cache ```/etc/nginx/conf.d/default.conf```
  ```
# wordpress multisite that handles cache purging and http/https (w http2)
###################################
## CACHE CFG      #################
###################################

fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=WORDPRESS:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$http_host$request_uri";
fastcgi_cache_use_stale error timeout invalid_header http_500;
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
add_header X-Cache $upstream_cache_status;

###################################

upstream php {  
    server wordpress:9000;
}

server {
    listen  80 default_server;
    server_name  example.com *.example.com;
    server_name_in_redirect off;

    access_log   /var/log/nginx/example.com.80.access.log;
    error_log    /var/log/nginx/example.com.80.error.log;

    root /var/www/html;
    client_max_body_size 128m;
    index index.php;

    include /etc/nginx/global/server.conf;
}

server {
    listen 443 http2 default_server;
    server_name example.com *.example.com;
    server_name_in_redirect off;

    access_log   /var/log/nginx/example.com.443.access.log;
    error_log    /var/log/nginx/example.com.443.error.log;

    root /var/www/html;
    client_max_body_size 128m;
    index index.php;

    include /etc/nginx/global/ssl.conf;
    include /etc/nginx/global/server.conf;
}
  ```

2. Specify a \***\_cache_purge** directive.
  - e.g. - WordPress Multisite FastCGI Cache Purge ```/etc/nginx/global/server.conf```
  ```
###################################
## CACHE EXCEPTIONS      ##########
###################################

set $skip_cache 0;

# POST requests and urls with a query string should always go to PHP
if ($request_method = POST) {
    set $skip_cache 1;
}   
if ($query_string != "") {
    set $skip_cache 1;
}   

# Do not cache uris containing the following segments
if ($request_uri ~* "/wp-admin/|/xmlrpc.php|wp-.*.php|/feed/|index.php|sitemap(_index)?.xml") {
    set $skip_cache 1;
}   

# Do not use the cache for logged in users or recent commenters
if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
    set $skip_cache 1;
}

###################################
## WORDPRESS LOCATIONS ############
###################################

location / {
    try_files $uri $uri/ /index.php?$args;
}

###################################
## FASTCGI PASS LOCATION   ########
###################################

location ~ \.php$ {
    try_files $uri =404; 
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PHP_VALUE "upload_max_filesize = 128m";
    fastcgi_param PHP_VALUE "post_max_size = 128m";
    fastcgi_pass wordpress:9000;
    fastcgi_cache_bypass $skip_cache;
    fastcgi_no_cache $skip_cache;
    fastcgi_cache WORDPRESS;
    fastcgi_cache_valid  60m;
}

###################################
## CACHE PURGE ####################
###################################

location ~ /purge(/.*) {
    fastcgi_cache_purge WORDPRESS "$scheme$request_method$http_host$1";
}

###################################

location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
    access_log off;	log_not_found off; expires max;
}

location = /favicon.ico { log_not_found off; access_log off; }
location = /robots.txt { access_log off; log_not_found off; }
location ~ /\. { deny  all; access_log off; log_not_found off; }
location ~* /(?:uploads|files)/.*\.php$ { deny all; access_log off; log_not_found off; }
  ```

