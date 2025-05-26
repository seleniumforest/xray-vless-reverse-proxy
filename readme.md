Xray config example: reverse proxy with vless, nginx, subdomains, fail2ban, mTLS. Used for bridging self-hosted services under NAT to public. Listens only 443 from outside

## file paths:

/etc/nginx/stream-enabled/proxy.conf 

/etc/nginx/nginx.conf

/etc/systemd/system/myapp.service

/etc/fail2ban/jail.d/nginx-badbasic.local

/etc/fail2ban/filter.d/nginx-badbasic.conf

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
sudo certbot certonly --standalone -d example -d qb.mydomain.com -d dufs.mydomain.com
```

### show subdomains 

```
openssl x509 -in /etc/letsencrypt/live/mydomain.com/fullchain.pem -text | grep DNS:
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


### jail check 

```
sudo fail2ban-client status
sudo fail2ban-client status nginx-badbasic
```

### unban 

```
sudo fail2ban-client set nginx-badbasic unbanip <ip>
sudo fail2ban-client unban --all
```

### generate client auth certs 

```
# 1. Создаём CA (root.pem, root.key)
openssl req -x509 -new -nodes -days 3650 -keyout root.key -out root.pem -subj "/CN=MyRootCA"

# 2. Серверный сертификат (server.crt, server.key)
openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr -subj "/CN=mydomain.com"
openssl x509 -req -in server.csr -CA root.pem -CAkey root.key -CAcreateserial -out server.crt -days 3650

# 3. Клиентский сертификат (client.crt, client.key)
openssl req -newkey rsa:2048 -nodes -keyout client.key -out client.csr -subj "/CN=myclient"
openssl x509 -req -in client.csr -CA root.pem -CAkey root.key -CAcreateserial -out client.crt -days 3650

```

```
client.crt — сертификат клиента, подписанный твоим CA (root.pem)

client.key — приватный ключ клиента

client.csr — запрос на сертификат клиента (можно удалить, если уже есть .crt)

server.crt — сертификат сервера (подписан root.pem)

server.key — приватный ключ сервера

server.csr — запрос на сертификат сервера (можно удалить)

root.pem — публичный сертификат корневого центра сертификации (CA), которым подписаны все остальные

root.key — приватный ключ CA

root.srl — служебный файл openssl, нужен только для подписи новых сертификатов (можно оставить рядом с root.key)
```

### create .p12 cert
```
openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12 -name "My Client Cert"
```