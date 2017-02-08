# nginx-cache-purge-http2

This is a Docker Nginx (v1.11.9) image built with [FRiCKLE's ngx_cache_purge module](https://github.com/FRiCKLE/ngx_cache_purge), the [Nginx's http_v2_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html) and [OpenSSL v1.0.2k library](https://www.openssl.org/) (providing HTTP/2 support via ALPN).

It's a merger of [procraft's nginx-purge-docker](https://github.com/procraft/nginx-purge-docker) and [Ehekatl's docker-nginx-http2](https://github.com/Ehekatl/docker-nginx-http2), which are built with the ngx_cache_purge module v2.4 and openssl v1.0.2 , respectively.

---

##How to use this image:

Use this image as a Dockerfile base (i.e.,```FROM stcox/nginx-cache-purge-http2```) for building a final image that's been configured to:

1. Purge content from FastCGI, proxy, SCGI and uWSGI caches, and/or
2. Enable HTTP/2 via OpenSSL ALPN.

Bake Nginx configuration files and SSL certificates directly into your image, statically with COPY, or use ```docker-entrypoint.sh``` sed/awk commands to configure Nginx dynamically on container startup.

---

**Enable Cache Purging:**

1. Configure a working cache (FastCGI, proxy, SCGI, or uWSGI).
  - e.g. - WordPress FastCGI Cache ```/etc/nginx/conf.d/default.conf```
  ```conf
# wordpress multisite

fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=WORDPRESS:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$http_host$request_uri";
fastcgi_cache_use_stale error timeout invalid_header http_500;
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;

add_header X-Cache $upstream_cache_status;

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

  include /etc/nginx/global/cache.conf;
  include /etc/nginx/global/locations.conf;
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
  include /etc/nginx/global/cache.conf;
  include /etc/nginx/global/locations.conf;
}
  ```

2. Specify a \***\_cache_purge** directive. 

_See also:_
- [Maximizing Python Performance with NGINX, Part 1: Web Serving and Caching](https://www.nginx.com/blog/maximizing-python-performance-with-nginx-parti-web-serving-and-caching/)
- [Content Caching with Nginx Plus](https://www.nginx.com/products/content-caching-nginx-plus/)

---
**Enable HTTP/2:**

1. Configure a server block specifying ```listen 443 http2``` directive.

_See also:_
- https://www.nginx.com/blog/nginx-1-9-5/
- https://nginx.org/en/docs/http/ngx_http_v2_module.html
- https://www.openssl.org/news/openssl-1.0.2-notes.html
- https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation


---
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
