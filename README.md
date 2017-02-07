# nginx-cache-purge-http2

Nginx (1.11.9) image supporting FastCGI ngx_cache_purge (2.4) and http2 via openssl (1.0.2k)

Based on [Ehekatl's docker-nginx-http2](https://github.com/Ehekatl/docker-nginx-http2) and [procraft's nginx-purge-docker](https://github.com/procraft/nginx-purge-docker).

This is a base image. Add nginx confs, certs and any other configuration to your top layer, e.g.:

```
# Dockerfile

FROM stcox/nginx-cache-purge-http2 # base image

COPY nginx.conf           /etc/nginx/nginx.conf
COPY default.conf         /etc/nginx/conf.d/default.conf
COPY ssl.conf             /etc/nginx/global/ssl.conf
COPY cache.conf           /etc/nginx/global/cache.conf
COPY proxy.conf           /etc/nginx/global/proxy.conf
COPY locations.conf       /etc/nginx/global/locations.conf
COPY docker-entrypoint.sh /entrypoint.sh

#ssl
COPY example.com.crt /etc/nginx/ssl/example.com.crt
COPY example.com.key /etc/nginx/ssl/example.com.key
COPY dhparam.pem     /etc/nginx/ssl/dhparam.pem

ENTRYPOINT ["/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```
