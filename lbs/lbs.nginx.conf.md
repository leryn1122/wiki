

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream backend {
  server 127.0.0.1:8080     weight=1 max_fails=5 fail_timeout=30s;
  server 127.0.0.2:8080     weight=1 max_fails=5 fail_timeout=30s;
  server 127.0.0.3:8080     weight=1 max_fails=5 fail_timeout=30s;
}

upstream frontend {
  server 127.0.0.11:8081     weight=1 max_fails=5 fail_timeout=30s;
  server 127.0.0.12:8081     weight=1 max_fails=5 fail_timeout=30s;
  server 127.0.0.13:8081     weight=1 max_fails=5 fail_timeout=30s;
  ip_hash;
}

server {
  listen      80;
  listen      443 ssl;
  server_name overture.leryn.top;
  
  if ($scheme = http) { return 301 https://$host$request_uri; }
  if ($host !~ (.*)leryn.top) { return 404; }
  
  ssl_protocols             TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;
  ssl_session_timeout       5m;
  ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
  ssl_certificate           /etc/tls/leryn.top/leryn.top.cer;
  ssl_certificate_key       /etc/tls/leryn.top/leryn.top.key;

  gzip               on;
  gzip_buffers       4 16k;
  gzip_comp_level    6;
  gzip_min_length    1k;
  gzip_types text/plain applcation/x-javascript application/javascript text/css application/xml text/javascript application/x-httpd-php; 
  gzip_static        on;
  gzip_vary          on;

  autoindex            on;
  autoindex_exact_size off;
  autoindex_localtime  on;
  charset              utf-8;

  chunked_transfer_encoding on;
  client_max_body_size   1G;
  tcp_nodelay            on;

  proxy_connect_timeout  3000;
  proxy_send_timeout     3000;
  proxy_read_timeout     3000;
  proxy_buffering        on;

  proxy_set_header       Host              $host;
  proxy_set_header       X-Real-IP         $remote_addr;
  proxy_set_header       X-Forwarded-For   $proxy_add_x_forwarded_for;
  proxy_set_header       X-Forwarded-Proto "https";
  
  location ^~ /api/ {
    proxy_pass http://backend/;
  }
  
  location ^~ /ws/ {
    #proxy_http_version     1.1;
    proxy_set_header       Upgrade         $http_upgrade;
    proxy_set_header       Connection      "Upgrade";
    proxy_pass http://backend/;
  }
  
  location / {
    proxy_set_header Access-Control-Allow-Credentials  true;
    proxy_set_header Access-Control-Allow-Origin       $host;
    proxy_set_header Access-Control-Allow-Headers      X-Requested-With;
    proxy_set_header Access-Control-Allow-Methods      GET,POST,PUT,DELETE,OPTIONS;

    add_header Access-Control-Allow-Credentials  true;
    add_header Access-Control-Allow-Origin       $host;
    add_header Access-Control-Allow-Headers      X-Requested-With;
    add_header Access-Control-Allow-Methods      GET,POST,PUT,DELETE,OPTIONS;
    
    proxy_pass http://frontend/;
  }
  
  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root   html;
  }
}
```
