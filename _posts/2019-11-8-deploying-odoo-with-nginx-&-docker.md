This document describes basic steps to set up Odoo in production or on an internet-facing server with Nginx and Docker

# Builtin server

* Server with 4 CPU, 8 Thread
* 60 concurrent users
* 60 users / 6 = 10 <- theorical number of worker needed
* (4 * 2) + 1 = 9 <- theorical maximal number of worker
* We’ll use 8 workers + 1 for cron. We’ll also use a monitoring system to measure cpu load, and check if it’s between 7 and 7.5 .
* RAM = 9 * ((0.8*150) + (0.2*1024)) ~= 3Go RAM for Odoo

in ```/config/odoo.conf:```

```
[options]
limit_memory_hard = 1677721600
limit_memory_soft = 629145600
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
max_cron_threads = 1
longpolling_port = 8072
workers = 8
```

## Configure Nginx as a Reverse Proxy

```$ sudo vi /etc/nginx/sites-enabled/odoo.example.com```

```
upstream odoo {
    server 127.0.0.1:8069;
}

server {
   listen      80 default;
   server_name odoo.example.com;

   access_log  /var/log/nginx/odoo.access.log;
   error_log   /var/log/nginx/odoo.error.log;

   proxy_buffers 16 64k;
   proxy_buffer_size 128k;

   location / {
       proxy_pass  http://odoo;
       proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
       proxy_redirect off;

       proxy_set_header    Host            $host;
       proxy_set_header    X-Real-IP       $remote_addr;
       proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header    X-Forwarded-Proto https;
   }

   location /longpolling {
        proxy_pass http://127.0.0.1:8072;
   }

   location ~* /web/static/ {
       proxy_cache_valid 200 60m;
       proxy_buffering on;
       expires 864000;
       proxy_pass http://odoo;
   }
}
```

```$ sudo ln -s /etc/nginx/sites-available/odoo.example.com /etc/nginx/sites-enabled/```

```$ sudo systemctl restart nginx```

# Docker Compose example

```
version: '2'
services:
  web:
    image: odoo:12.0
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./config:/etc/odoo
      - ./:/mnt/extra-addons
  db:
    image: postgres:10
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - odoo-db-data:/var/lib/postgresql/data/pgdata
volumes:
  odoo-web-data:
  odoo-db-data:
```

To start your Odoo instance, go in the directory of the docker-compose.yml file you created from the previous examples and type:

```$ docker-compose up -d ```
