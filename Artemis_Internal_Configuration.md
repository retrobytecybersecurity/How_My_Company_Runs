# How_My_Company_Runs
Details on how the entire company is built internally



# Artemis Subdomain Built
SUBDOMAIN -> is connected to Cloudflare

Log into Cloudflare → select retrobytecybersecurity.org → DNS → Add record:
```
Type:    A
Name:    artemis
Content: your-linode-ip
Proxy:   OFF (grey cloud, not orange)
TTL:     Auto
```

This creates artemis.retrobytecybersecurity.org pointing to your Linode. Proxy must be OFF — Cloudflare's proxy doesn't support arbitrary ports and would interfere with your SSL cert setup


sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d artemis.retrobytecybersecurity.org


sudo vim /etc/nginx/sites-enabled/artemis

```
server {
    listen 80;
    server_name artemis.retrobytecybersecurity.org;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name artemis.retrobytecybersecurity.org;

    ssl_certificate     /etc/letsencrypt/live/artemis.retrobytecybersecurity.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/artemis.retrobytecybersecurity.org/privkey.pem;

    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_read_timeout 300;
        proxy_buffering    off;
    }
}
```

sudo nginx -t
sudo systemctl reload nginx

# In Linode Cloud Firewall — add inbound rule:
# TCP port 80, all sources
