Xray config example: reverse proxy with vless, nginx, subdomains. Used for bridging my self-hosted services under NAT to public. 

## file paths:

/etc/nginx/stream-enabled/proxy.conf 

/etc/nginx/nginx.conf

/etc/systemd/system/myapp.service

## sources:

https://habr.com/ru/articles/774838/

https://coyotle.ru/posts/xray-proxy/

https://coyotle.ru/posts/xray-and-nginx/

## useful commands:

### close incoming port

```
sudo apt install iptables-persistent
sudo iptables -A INPUT -p tcp -s 127.0.0.1 --dport 8080 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP
sudo iptables -L -n | grep 8080
sudo netfilter-persistent save
```

### generate certs for subdomains

```
sudo certbot certonly --standalone -d example -d qb.example.com -d dufs.example.com
```

### show subdomains 

```
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -text | grep DNS:
```

###  create systemd service 

```
[Unit]
Description=XRay
After=network.target

[Service]
Type=simple
ExecStart=/root/xray run -c=/root/config.json
Restart=on-failure
User=root
Group=root

[Install]
WantedBy=multi-user.target

```

```
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
sudo systemctl status myapp.service
```

### add logging to xray

```

{
"log": {"logLevel": "debug"},
"reverse":
"bridges":
"outbounds":
}
```