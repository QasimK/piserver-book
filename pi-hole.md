# Pi-Hole

Prereqs: Nginx

Pi-Hole will allow us to, optionally, secure the DNS, blocks ads, monitor DNS queries, and do all of that for the entire LAN.

Unfortunately it doesn't offer DNS-over-TLS or DNS-over-HTTPS out of the box, so we're not gonna bother with it until it does.

[https://github.com/notasausage/pi-hole-unbound-wireguard](https://github.com/notasausage/pi-hole-unbound-wireguard)

## Security

* TODO: PHP open\_basedir directive

* TODO: PiHole-FTLDNS has a local statistics server at `localhost:4711` usable with telnet which cannot be disabled. \(It is also not possible to set the configuration port to an invalid value.

* TODO: Cloudflared has a local metrics server at `loocalhost:38083/metrics` which cannot be disabled.

## Installation

We will install Pi-Hole with the admin dashboard accessible via our Nginx reverse proxy. There are [separate instructions](/web-server-nginx.md) to install Nginx.

```console
# The existing DNS resolver on Port 53 must be disabled
systemctl disable --now systemd-resolved

# As non-root user install Pi-Hole
yay -S --needed pi-hole-server

pacmatic -S --needed php-fpm php-sqlite
```

Enable necessary PHP extensions in `/etc/php/php.ini`

```
[...]
extension=pdo_sqlite
[...]
extension=sockets
extension=sqlite3
[...]
```

**TODO: open\_basedir**

Password Protect the admin interface:

```
pihole -a -p
```

Modify `/etc/pihole/pihole-FTL.conf`:

```ini
# Optimise for storage lifetime
DBINTERVAL=60.0

# Reduce length of time queries are stored
MAXDBDAYS=30
```

Add the Nginx config to `/etc/nginx/sites-available/pihole.conf`:

```nginx
server {
    listen [::]:443 ssl http2 ipv6only=off;
    allow 192.168.1.0/24;
    deny all;

    root /srv/http/pihole;
    server_name pihole.piserver.local;
    autoindex off;

    proxy_intercept_errors on;
    error_page 404 /pihole/index.php;

    location / {
        return 301 /admin;
    }

    location ~ \.php$ {
        include fastcgi.conf;
        fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_param SERVER_NAME $host;
        fastcgi_param FQDN true;
        # Mitigate https://httpoxy.org/ vulnerabilities
        fastcgi_param HTTP_PROXY "";
    }   

    location /pihole {
        root /srv/http/pihole;
    }

    location /admin {
        root /srv/http/pihole;
        index index.php;
    }
}
```

Now enable the admin dashboard:

```console
systemctl enable --now php-fpm.server
```

TODO: Backup - /etc/pihole/setupVars.conf, /etc/php/php.ini, nginx stuff, /etc/pihole/pihole-FTL.conf.

TODO: Nginx gzip config into Nginx page.

### Personal VPN Configuration

Modify `/etc/pihole/setupVars.conf`, ensure static LAN IP addresses are used for the IPv4 and IPv6 addresses:

```ini
PIHOLE_INTERFACE=eth0
# DNSMASQ_LISTENING=local overrides PIHOLE_INTERFACE
DNSMASQ_LISTENING=local
IPV4_ADDRESS=...
IPV6_ADDRESS=...
```

However, the above DNSMASQ\_LISTENING does not reliablely cause dnsmasq to listen on all local interfaces, so we override `/etc/dnsmasq.d/99-pivpn.conf`:

```ini
interface=eth0
interface=pivpn
```

Modify Nginx config to allow admin dashboard connections from the VPN:

```nginx
    #Allow our Personal VPN
    allow 10.0.0.0/24;
```

### Cloudflared DNS-over-HTTPS

First install `cloudflared`:

```console
$ yay -S --needed cloudflared-bin
```

Create the configuration `/etc/cloudflared/cloudflared.conf`

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

> The IPv6 upstream addresses should be removed if we do not have an IPv6-capable connection.

Override service to ensure it has the necessary permissions to run using an unprivileged user `systemctl edit cloudflared@cloudflared`:

```ini
[Service]
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
```

Enable and start the service:

```console
# systemctl enable --now cloudflared@cloudflared
```

> This includes a metrics server that cannot be disabled: `curl 127.0.0.1:38083/metrics`

Now modify Pi-Hole to use this DNS resolver.

First modify `/etc/pihole/setupVars.conf`:

```ini
PIHOLE_DNS_1=127.0.0.1#5053
```

Then modify `/etc/dnsmasq.d/98-cloudflared.conf`:

```ini
server=127.0.0.1#5053
```

## Backup

Add the following files to the backup script:

```
    /etc/pihole/setupVars.conf \
    /etc/pihole/pihole-FTL.conf \
    /etc/dnsmasq.d/98-cloudflared.conf \
    /etc/dnsmasq.d/99-pivpn.conf \
    /etc/php/php.ini \
    /etc/systemd/system/cloudflared@cloudflared.service.d\override.conf \
    /etc/systemd/system/multi-user.target.wants/cloudflared@cloudflared.service \
    /etc/systemd/system/multi-user.target.wants/php-fpm.service \
    /etc/systemd/system/multi-user.target.wants/pihole-FTL.service \
```



