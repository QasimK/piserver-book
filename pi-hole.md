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

Password Protect the admin interface:

```
$ pihole -a -p
```

Modify `/etc/pihole/pihole-FTL.conf`:

```ini
# Optimise for storage lifetime
DBINTERVAL=60.0

# Reduce length of time queries are stored
MAXDBDAYS=30
```

nginx conf.: \(cp /usr/share/pihole/configs/nginx.example.conf pihole.conf\)

```
TODO: COPY AND PASTE!
```

**systemctl enable --now php-fpm.service**

Modify /etc/pihole/setupVars.conf

`# Overrides PIHOLE_INTERFACE`

`DNSMASQ_LISTENING=local`

Except that ^ doesn't seem to work reliably. So override 

TODO: Backup - /etc/pihole/setupVars.conf, /etc/php/php.ini, nginx stuff, /etc/pihole/pihole-FTL.conf.

Security issues:

PiHole FTL has open port on localhost:4711 for statistics \(cannot just set to invalid value. Maybe mask it\). This can be used with telnet.

TODO: Nginx gzip config.

### Personal VPN Configuration

Modify `/etc/pihole/setupVars.conf`, ensure static LAN IP addresses are used for the IPv4 and IPv6 addresses:

```ini
PIHOLE_INTERFACE=eth0
# DNSMASQ_LISTENING=local overrides PIHOLE_INTERFACE
DNSMASQ_LISTENING=local
IPV4_ADDRESS=...
IPV6_ADDRESS=...
```

However, the above DNSMASQ\_LISTENING does not reliablely cause dnsmasq to listen on all local interfaces, so we override `/etc/dnsmasq.d/99-override.conf`:

```ini
interface=eth0
interface=pivpn
```

## cloudflared adblocker with pihole

First install `cloudflared`:

`yay -S --needed cloudflared-bin`

Create example conf in /etc/cloudflared/pivpn.conf

```yaml
proxy-dns: true
proxy-dns-upstream:
 - https://1.0.0.1/.well-known/dns-query
 - https://1.1.1.1/.well-known/dns-query
 - https://2606:4700:4700::1111/.well-known/dns-query
 - https://2606:4700:4700::1001/.well-known/dns-query
proxy-dns-port: 5053
proxy-dns-address: 127.0.0.1
```

Override service

```ini
[Service]
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
```

Modify /etc/pihole/setupVars.conf

PIHOLE\_DNS 1 with 127.0.0.1\#5053

