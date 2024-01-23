---
layout: default
title: Generate Let's Encrypt Certificate Using Certbot
parent: Linux
nav_order: 4
---

## How to Generate Let's Encrypt Certificate Using Certbot

**Setup a new VM to install certbot**
```shell
#install certbot
sudo apt install certbot
```

```shell
sudo certbot certonly --standalone --preferred-challenges http -d cache-server.test.local -m <your-email> --agree-tos
```

**NOTE:** Certbot will place the generated certificates in the /etc/letsencrypt/live/cache-server-test-local/ directory. Update your web server configuration (e.g., Nginx, Apache) to use these certificates.

### For Nginx:
```shell
server {
    listen 443 ssl;
    server_name cache-server.test.local;

    ssl_certificate /etc/letsencrypt/live/cache-server-test-local/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cache-server-test-local/privkey.pem;

    # Increase the client_max_body_size to 100 MB
    client_max_body_size 100M;

    location / {
        proxy_pass http://cache-server.test.local:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Preserve the Authorization header
        proxy_set_header Authorization $http_authorization;

        # Add any additional proxy headers if needed
        # ...

        # Adjust the proxy timeouts if needed
        proxy_connect_timeout       600;
        proxy_send_timeout          600;
        proxy_read_timeout          600;
        send_timeout                600;
    }
}
```