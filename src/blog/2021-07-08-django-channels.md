---
title: Deploy Django + Channels website with nginx and daphne server on ubuntu
author: Pratik Kute
date: 2021-07-08
image: /assets/blog/django-channels.webp
tags: ["post", "featured"]
imageAlt: Django + Channels deployment
description: Deploy Django + Channels website with nginx server on ubuntu
---

# Deploy Django + Channels website with nginx and daphne server on ubuntu

## Get all files here [Github repo](https://github.com/pratikkute/deploy_new_websiite).

## Specifications

1. Ubuntu 20.04
2. Django Channels 3
3. Redis Install and Config
4. ASGI for Hosting Django Channels
5. Deploying Django Channels with Daphne using supervisor
   1. Starting the daphne service
   2. Writing a supervisor script that tells daphne to start
6. Configure Nginx
7. HTTPS with [letsencrypt](https://letsencrypt.org/)

## basic apt reuired

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install python3.9 -y
sudo apt-get install python3.9-dev python3.9-venv -y
sudo apt-get install python3-pip -y
sudo apt install nginx supervisor
```

## requirement.txt

``` txt
django
channels
channels_redis
```
## Start Django project in virtual env
``` bash
cd ~
mkdir venv
python3.9 -m venv ~/venv/.
source ~/venv/bin/activate
pip install setproctitle
pip install -r ~/requirements.txt
django-admin.py startproject PROJECT-NAME
cd ~/PROJECT-NAME
mkdir media static templates
python manage.py startapp home
python manage.py migrate
sudo adduser $USER www-data
sudo chown -R :www-data ~/PROJECT-NAME/
sudo chown :www-data ~/PROJECT-NAME/db.sqlite3
sudo chmod -R 775 ~/venv/
sudo chmod -R 775 ~/PROJECT-NAME/
```


## Install and Setup Redis

Redis is used as a kind of "messaging queue" for django channels. Read more about it here [channel_layers](https://channels.readthedocs.io/en/stable/topics/channel_layers.html?highlight=redis#redis-channel-layer)

``` bash
sudo apt install redis-server
```

Navigate to `/etc/redis/`

open `redis.conf`

`CTRL+F` to find 'supervised no'

change 'supervised no' to 'supervised systemd'

`SAVE`

``` bash
sudo systemctl restart redis.service
sudo systemctl status redis
```

### Add "CHANNEL_LAYERS" & "ASGI_APPLICATION" to installed app in settings

```py
ASGI_APPLICATION = "PROJECT-FOLDER.asgi.application"

CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("redis-server-name", 6379)],
        },
    },
}
```

### Edit asgi.py

```py
import os

from django.core.asgi import get_asgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "PROJECT-FOLDER.settings")
django_asgi_app = get_asgi_application()

from channels.routing import ProtocolTypeRouter, URLRouter

from channels.auth import AuthMiddlewareStack
from django.urls import path

from chat.consumers import ChatConsumer

application = ProtocolTypeRouter(
    {
        "http": django_asgi_app,
        "websocket": AuthMiddlewareStack(
            URLRouter(
                [
                    path("", ChatConsumer.as_asgi()),
                ]
            )
        ),
    }
)

```

## Deploying Django Channels with Daphne [Ref](https://channels.readthedocs.io/en/stable/deploying.html)


Now, you will need to create the supervisor configuration file (often located in /etc/supervisor/conf.d/ - here, we’re making Supervisor listen on the TCP port and then handing that socket off to the child processes so they can all share the same bound port:

### supervior.conf

``` editorconfig
[fcgi-program:asgi]
socket=tcp://localhost:8001
directory=/home/ubuntu/PROJECT-FOLDER
command=/home/ubuntu/venv/bin/daphne -u /home/ubuntu/PROJECT-FOLDER/run/daphne%(process_num)d.sock --fd 0 --access-log - --proxy-headers PROJECT-FOLDER.asgi:application
numprocs=4
process_name=asgi%(process_num)d
autostart=true
autorestart=true
stdout_logfile=/home/ubuntu/logs/daphne-error.log
redirect_stderr=true
```

## Create the run directory for the sockets referenced in the supervisor configuration file.

```bash
sudo mkdir /home/ubuntu/PROJECT-FOLDER/run/
sudo mkdir /home/ubuntu/logs/
sudo chown <user>.<group> /home/ubuntu/PROJECT-FOLDER/run/
sudo chmod -R 775 ~/PROJECT-FOLDER/
```

## Have supervisor reread and update its jobs:

```bash
sudo supervisorctl reread
sudo supervisorctl update
```

## Configure Nginx

We need to tell nginx to allow websocket data to move through port 8001.

Navigate to `/etc/nginx/sites-available`

Update `default` file

```nginx
upstream daphne_server {
    server localhost:8001;
}

server {

    listen 80 default_server;
    listen [::]:80 default_server;
    server_name <DOMAIN>;

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /home/ubuntu/logs/nginx-access.log;
    error_log /home/ubuntu/logs/nginx-error.log;

    location /static/ {
        alias /home/ubuntu/PROJECT-FOLDER/static/;
    }

    location /media/ {
        alias /home/ubuntu/PROJECT-FOLDER/media/;
    }

    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
        proxy_pass http://daphne_server;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
    }

}

```



## HTTPS (If you have a domain registered and it's working)

**Do not do this step unless you're able to visit your website using the custom domain.** See [How do you know it's working?](How-do-you-know-its-working?)

#### Install certbot

HTTPS is a little more difficult to set up when using Django Channels. Nginx and Daphne require some extra configuring.

```bash
sudo apt install certbot python3-certbot-nginx
sudo systemctl reload nginx
```

Make sure nginx configuration is still good.

``` bash
sudo nginx -t
```

#### Allow HTTPS through firewall

`sudo ufw allow 'Nginx Full'`

`sudo ufw delete allow 'Nginx HTTP'` Block standard HTTP

#### Obtain SSL certificate

`sudo certbot --nginx -d <your-domain.whatever> -d www.<your-domain.whatever>`

For eg:

``` bash
sudo certbot --nginx -d demo.xyz -d www.demo.xyz
```

#### Verifying Certbot Auto-Renewal

``` bash
sudo systemctl status certbot.timer
```

#### Test renewal process
``` bash
sudo certbot renew --dry-run
```
### Update nginx config

Navigate to `/etc/nginx/sites-available`

Update `default` file for the cert

```nginx
upstream daphne_server {
    server localhost:8001;
}

server {

    listen 80 default_server;
    listen [::]:80 default_server;
    listen 443 ssl; # managed by Certbot
    server_name <DOMAIN>;


    # RSA certificate
    ssl_certificate /etc/letsencrypt/live/<DOMAIN>/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/<DOMAIN>/privkey.pem; # managed by Certbot

    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

    Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /home/ubuntu/logs/nginx-access.log;
    error_log /home/ubuntu/logs/nginx-error.log;

    location /static/ {
        alias /home/ubuntu/PROJECT-FOLDER/static/;
    }

    location /media/ {
        alias /home/ubuntu/PROJECT-FOLDER/media/;
    }

    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
        proxy_pass http://daphne_server;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
    }

}

```

## Check and restart nginx and daphne server


``` bash
sudo supervisorctl restart asgi
sudo supervisorctl status
sudo nginx -t && sudo systemctl restart nginx
```
