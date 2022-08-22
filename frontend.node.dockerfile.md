
<a name="SReK5"></a>
## Dockerfile

```dockerfile
FROM node:16-slim as build

WORKDIR /opt

COPY .npmrc ~
COPY . .

RUN npm install pnpm -g \
  && pnpm install \
  && pnpm build

FROM nginx:1.21.1 as publish

COPY --from=build /opt/nginx.conf /etc/nginx/nginx.conf.d/website.conf
COPY --from=build /opt/dist/      /usr/share/nginx/html/

EXPOSE 80/tcp

ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
<a name="IuqMd"></a>
## nginx.conf

```
# Default nginx configuration for docker.

http {

  sendfile            on;
  tcp_nopush          on;
  tcp_nodelay         on;
  keepalive_timeout   65;
  
  types_hash_max_size 2048;
  include             /etc/nginx/mime.types;

  # Customized
  # Gzip for static file.
  gzip               on;
  gzip_buffers       4 16k;
  gzip_comp_level    6;
  gzip_min_length    1k;
  gzip_types text/plain applcation/x-javascript application/javascript text/css text/xml application/xml  application/xml+rss text/javascript application/x-httpd-php image/svg+xml;
  gzip_static        on;
  gzip_vary          on;
  gzip_proxied       any;
  gzip_http_version  1.0;
  gzip_disable       "MSIE [1-6]\.";

  # Brotli for static file.
  brotli             on;
  brotli_comp_level  6;
  brotli_buffers     16 8k;
  brotli_min_length  20;
  brotli_types text/plain applcation/x-javascript application/javascript text/css text/xml application/xml  application/xml+rss text/javascript application/x-httpd-php image/svg+xml;

  server {
    listen       80;
    server_name  localhost;
    location / {
      root   /usr/share/nginx/html;
      index  index.html index.htm;
      try_files $uri $uri/ /index.html;
    }
  }
}
```
