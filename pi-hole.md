# Pi-Hole

Prereqs: Nginx

Pi-Hole will allow us to, optionally, secure the DNS, blocks ads, monitor DNS queries, and do all of that for the entire LAN.

Unfortunately it doesn't offer DNS-over-TLS or DNS-over-HTTPS out of the box, so we're not gonna bother with it until it does.

[https://github.com/notasausage/pi-hole-unbound-wireguard](https://github.com/notasausage/pi-hole-unbound-wireguard)

## Steps

We will use Nginx.

```
systemctl disable --now systemd-resolved

# As user not root
yay -S --needed pi-hole-server

pacmatic -S --needed php-fpm php-sqlite
# pacmatic -S --needed nginx-mainline
# configured separately.
```

Enable extensions in /etc/php/php.ini

```
[...]
extension=pdo_sqlite
[...]
extension=sockets
extension=sqlite3
[...]
```

TODO: open\_basedir

password-protect interface:

pihole -a -p

nginx conf.: \(cp /usr/share/pihole/configs/nginx.example.conf pihole.conf\)

```
TODO: COPY AND PASTE!
```

systemctl enable --now php-fpm.service

Modify /etc/pihole/setupVars.conf

\# Overrides PIHOLE\_INTERFACE

DNSMASQ\_LISTENING=local

Backup - /etc/pihole/setupVars.conf, /etc/php/php.ini, nginx stuff

## cloudflared adblocker \(not with PiHole\) - does not work

yay -S --needed cloudflared-bin

Create example conf in /etc/cloudflared/pivpn.conf

```
proxy-dns: true
proxy-dns-upstream:
 - https://1.0.0.1/.well-known/dns-query
 - https://1.1.1.1/.well-known/dns-query
 - https://2606:4700:4700::1111/.well-known/dns-query
 - https://2606:4700:4700::1001/.well-known/dns-query
proxy-dns-port: 53
proxy-dns-address: 0.0.0.0
```

Override service

```
[Service]
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
```

Modify /etc/pihole/setupVars.conf

PIHOLE\_DNS 1 with 127.0.0.1\#5053

