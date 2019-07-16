 ### 要开启多work 
 #### 首先` proxy_mode= True` 需要打开并且配置 `longpolling_port = 8072 ` 其次才是`works = 4`
 #### works 个数和核心数比例  N*2 + 1
 
 
 
 
 ```conf
 
 upstream odoo8 {
 server 127.0.0.1:8069 weight=1 fail_timeout=0;
}

upstream odoo8im {
  server 127.0.0.1:8072;
}

  server {
    listen 80;
    server_name hesai.com;
    #access_log /home/yuan/odoo/nginx.log;
    proxy_set_header "Host" $host:8069;

    # Specifies the maximum accepted body size of a client request,
    # as indicated by the request header Content-Length.
    client_max_body_size 200m;

    # increase proxy buffer to handle some OpenERP web requests
    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    #general proxy settings
    # force timeouts if the backend dies
    proxy_connect_timeout 600s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

    # set headers
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;

    # Let the OpenERP web service know that we’re using HTTPS, otherwise
    # it will generate URL using http:// and not https://
    proxy_set_header X-Forwarded-Proto https;

    # by default, do not forward anything
    proxy_redirect off;
    proxy_buffering off;

    location /longpolling/  {
       proxy_pass http://odoo8im;

  }
    location /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo8;
    }
    location / {
        proxy_pass http://odoo8;
    }
    access_log off;
    log_not_found off;

  }
```
