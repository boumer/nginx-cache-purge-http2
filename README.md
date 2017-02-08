# nginx-cache-purge-http2

This is an Nginx (v1.11.9) image built with support for FastCGI ngx_cache_purge (v2.4) and HTTP/2 via OpenSSL ALPN (v1.0.2k).

It is a mashup based on [Ehekatl's docker-nginx-http2](https://github.com/Ehekatl/docker-nginx-http2) and [procraft's nginx-purge-docker](https://github.com/procraft/nginx-purge-docker), which are built with [openssl v1.0.2](https://www.openssl.org/) and [FRiCKLE's ngx_cache_purge v2.4](https://github.com/FRiCKLE/ngx_cache_purge) respectively.

Use this image as a base (```FROM stcox/nginx-cache-purge-http2```) to build a final image configured to:

1. Purge content from FastCGI, proxy, SCGI and uWSGI caches, and/or
2. Enable HTTP/2 via SSL.

You can bake Nginx conf files and SSL certificates into your image with COPY, or use docker-entrypoint.sh to configure Nginx on startup.

**Enable Cache Purging:** Configure proxy, FastCGI, SCGI, or uWSGI caching and specify a *_cache_purge _* directive. 

_See:_
- [Maximizing Python Performance with NGINX, Part 1: Web Serving and Caching](https://www.nginx.com/blog/maximizing-python-performance-with-nginx-parti-web-serving-and-caching/)
- [Content Caching with Nginx Plus](https://www.nginx.com/products/content-caching-nginx-plus/)


**Enable HTTP/2:** provide an nginx conf specifying a ```listen 443 http2 default_server``` directive.

_See:_
- https://www.nginx.com/blog/nginx-1-9-5/
- https://nginx.org/en/docs/http/ngx_http_v2_module.html
- https://www.openssl.org/news/openssl-1.0.2-notes.html
- https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation

```
# Dockerfile Example

FROM stcox/nginx-cache-purge-http2 # base image

COPY nginx.conf           /etc/nginx/nginx.conf
COPY default.conf         /etc/nginx/conf.d/default.conf
COPY docker-entrypoint.sh /entrypoint.sh

#ssl
COPY example.com.crt /etc/nginx/ssl/example.com.crt
COPY example.com.key /etc/nginx/ssl/example.com.key
COPY dhparam.pem     /etc/nginx/ssl/dhparam.pem

ENTRYPOINT ["/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
```
