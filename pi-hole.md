# Pi-Hole

Pre-requisites: Nginx

Pi-Hole will allow us to \(optionally\) secure DNS queries, blocks ads, monitor DNS queries, and do all of that for the entire LAN and Personal VPN connections.

In order to use this, individual devices' DNS configuration can be set to the PiServer's IP address, or the home router's DHCP/DNS can be configured to provide the PiServer's IP address to all devices on the LAN.

**Warning**:

* If the PiServer stops working, the internet will basically stop working for everyone on the LAN. The only way around this is to have a second - identical - Pi-Hole device, or to be really prepared to do a backup and restore process including having backup hardware ready to go.
* The ad-blocking may cause some things to not work for some people that they want to work. Therefore, **TODO: a disable button** can be shared with everyone.

## Security

* TODO: PHP `open_basedir` directive

* TODO: PiHole-FTLDNS has a local statistics server at `localhost:4711` usable with telnet which cannot be disabled. \(It is also not possible to set the configuration port to an invalid value.

* TODO: Cloudflared has a local metrics server at `loocalhost:38083/metrics` which cannot be disabled.

## Installation

We will install Pi-Hole with the admin dashboard accessible via our Nginx reverse proxy. There are [separate instructions](/web-server-nginx.md) to install Nginx.

```console
# The existing DNS resolver on Port 53 must be disabled
systemctl disable --now systemd-resolved

# As a non-root user install Pi-Hole from AUR
yay -S --needed pi-hole-server

# Install PHP for the web admin dashboard
pacmatic -S --needed php-fpm php-sqlite
```

Enable necessary PHP extensions by uncommenting the following lines in `/etc/php/php.ini`

```
[...]
extension=pdo_sqlite
[...]
extension=sockets
extension=sqlite3
[...]
```

TODO: memory-limit = 256M to fix possible query log errors

**TODO: open\_basedir for additional security.**

Password Protect the admin interface:

```
pihole -a -p
```

Modify `/etc/pihole/pihole-FTL.conf`:

```ini
# Optimise for storage lifetime
DBINTERVAL=60.0

# Reduce length of time individual DNS queries are stored
MAXDBDAYS=30
```

Now enable the ad-blocking DNS server:

```
systemctl enable --now pihole-FTL
```

TEMP FIXES:

* logrotate does not work - need chown root:root /etc/pihole/logrotate

**FIX**

usermod -aG pihole http

Give http full access to pihole

### Admin Dashboard

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

We also need to allow the dashboard access to certain files because the systemd service is hardened by default. Override the php-fpm config `systemctl edit php-fpm.service`:

TODO: Harden further, remove unnecessary

```
[Service]
ReadWritePaths = /srv/http/pihole
ReadWritePaths = /run/pihole-ftl/pihole-FTL.port
ReadWritePaths = /run/log/pihole/pihole.log
ReadWritePaths = /run/log/pihole-ftl/pihole-FTL.log
ReadWritePaths = /etc/pihole
ReadWritePaths = /etc/hosts
ReadWritePaths = /etc/hostname
ReadWritePaths = /etc/dnsmasq.d/
ReadWritePaths = /proc/meminfo
ReadWritePaths = /proc/cpuinfo
ReadWritePaths = /sys/class/thermal/thermal_zone0/temp
ReadWritePaths = /tmp
```

Now enable the admin dashboard:

```console
systemctl enable --now php-fpm.server
```

TODO: Nginx gzip config into Nginx page. Move Nginx config to Nginx page.

TODO: pihole updateGravity - update adblock lists cron.

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

Now on the VPN we can specify the DNS server to be PiServer's LAN IP address.

Additionally, we can modify Nginx config to allow admin dashboard connections from the VPN:

```nginx
    #Allow our Personal VPN
    allow 10.0.0.0/24;
```

### Cloudflared DNS-over-HTTPS

We can use Cloudflare's encrypted DNS-over-HTTPS service instead of our ISP's cleartext DNS servers.

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

## Conclusion

We have set up Pi-Hole which allows us to:

* block ads for everyone on the LAN, and
* block ads for everyone using the Personal VPN, and
* use DNS-over-HTTPS using cloudflared, and
* securely access the password-protected dashboard over the LAN and Personal VPN.



