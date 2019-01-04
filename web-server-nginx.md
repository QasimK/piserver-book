# Web Server \(with Nginx\)

A web server allows us to securely host web applications over the internet, or on the local LAN.

We can use Nginx to serve other applications that we have set up using TLS encryption over the local network and/or over the internet on our domain. In addition, we can set up basic username/password authentication for any web services without authentication \(e.g. [**System Monitoring \(Netdata\)**](/system-monitoring-netdata.md)\).

\(Alternatives might include: Apache, Lighttpd, or Caddy.\)

## Install

* [ ] TODO: Ciphers suitable for RPi

```
sudo pacmatic -S --needed nginx
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048 creates=/etc/ssl/certs/dhparam.pem
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

Edit `/etc/nginx/nginx.conf`

```ini
#user html;
worker_processes  1;  

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  65;

    include /etc/nginx/conf.d/*.conf;
    include sites-enabled/*;
}
```

> We generate fresh Diffie-Hellman parameters. This is an important security step, though it takes a while.
>
> Normally we would suggest 2048-bit, but the Raspberry Pi is a little underpowered :\) It takes fooorrrreevverrr just to generate the 2048-bit file.

## Configuration

We will enable a global redirect from HTTP -&gt; HTTPS `/etc/nginx/sites-available/default.conf`

```nginx
# Redirect all HTTP -> HTTPS
server {
    listen [::]:80 default_server ipv6only=off;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$host$request_uri;
}
```

We will add a couple of useful configuration helpers inside `/etc/nginx/conf.d/`

```nginx
# tls.conf
# Configure TLS security (requires setting parameters in server blocks to activate)

# USAGE:
# You must generate a dhparam.pem file.
# You must set ssl_certificate, ssl_certificate_key, and ssl_trusted_certificate.
# You must NOT use add_header inside your server {} block at all.

# 1MB = 4000 sessions
ssl_session_cache shared:SSL:1M;
ssl_session_timeout 1d;
# Session Tickets are a privacy leak
ssl_session_tickets off;

# Secure SSL config - https://mozilla.github.io/server-side-tls/ssl-config-generator/
ssl_protocols TLSv1.2 TLSv1.3;
ssl_dhparam /etc/ssl/certs/dhparam.pem;  # Must generate this manually
ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers on;

# SSL OSCP Stapling
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4;

# Force SSL to this domain (+subdomains) for 6 months (+ preload list)
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" always;
```

```nginx
# security_headers.conf
# Ensure HTTP headers that are important for security are set.
# We set some defaults which can be overridden by the upstream server.

# USAGE:
# You must use proxy_hide_header/uwsgi_hide_header to hide the upstream
#   headers for all headers that we list below.
# You must NOT use add_header inside your server {} block at all.

map $upstream_http_referrer_policy $referrer_policy {
    default $upstream_http_referrer_policy;
    '' "strict-origin-when-cross-origin";
}

map $upstream_http_x_content_type_options $x_content_type_options {
    default $upstream_http_x_content_type_options;
    '' "nosniff";
}

map $upstream_http_x_frame_options $x_frame_options {
    default $upstream_http_x_frame_options;
    '' "DENY";
}

map $upstream_http_x_xss_protection $x_xss_protection {
    default $upstream_http_x_xss_protection;
    '' "1; mode=block";
}

add_header Referrer-Policy $referrer_policy always;
add_header X-Content-Type-Options $x_content_type_options always;
add_header X-Frame-Options $x_frame_options always;
add_header X-XSS-Protection $x_xss_protection always;
```

We will modify `/etc/nginx/nginx.conf` slightly.

Also: Add ``tcpnopush on; under `sendfile on` This is an optimisation.``

## LAN-specific Configuration

LAN applications will be served at `piserver.local`. TLS encryption will be done with self-signed certificate which must be installed on your devices.

The most significant field is Common Name \(CN\) which should be set to `piserver.local`.

```
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

We create a `/etc/nginx/snippets/self-signed-cert.conf`

```
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```

Install the crt on your devices...

* [ ] Appendix this.

Linux:

```
scp piserver.local:/etc/ssl/certs/nginx-selfsigned.crt ~
sudo chmod 644 nginx-selfsigned.crt
sudo mv nginx-selfsigned.crt /usr/share/ca-certificates/trust-source/anchors/piserver-selfsigned.crt

# Arch Linux
sudo trust extract-compat

# Ubuntu
sudo update-ca-certificates
```

## Internet-specific Configuration

WWW applications will be served at `<DOMAIN>`. TLS encryption will be done using [Letsencrypt](https://letsencrypt.org/) which provides free certificates.

## Backup

We do not backup the self-signed certificate.

```
    /etc/nginx/nginx.conf \
    /etc/nginx/conf.d/ \
    /etc/nginx/snippets/ \
    /etc/nginx/sites-available/ \
    /etc/nginx/sites-enabled/ \
```

### Restore

Reinstall Nginx section - dhparam, certifications.

Gzip files again for each application again.

## Applications

### Transmission

We [**configured transmission**](/torrent-transmission.md) to run a web socket to `/run/transmission.sock`.

```nginx
# transmission.conf

upstream transmission {
    server unix:/run/transmission.sock;
}

server {
    listen 443 ssl http2  i;
    listen [::]:443 ssl http2;
    server_name piserver.local;
    charset utf-8;

    # Check if this certificate is really served for this server_name
    # http://serverfault.com/questions/578648/properly-setting-up-a-default-nginx-server-for-https
    if ($host != $server_name) {
        return 444;
    }

    include snippets/self-signed-cert.conf;

    location ~ ^/transmission/web/(style|images|javascript)/ {
        root /usr/share/;
        disable_symlinks if_not_owner;  # Extra-security
        gzip_static on; 

        access_log off;
        open_file_cache         max=100;
        open_file_cache_errors  on; 
    }

    location ~ ^/transmission/ {
        proxy_pass http://transmission;
        proxy_redirect http://127.0.0.1:9091 https://piserver.local;
        proxy_read_timeout 60; 
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        # Correct handling of fallbacks for HTTP headers
        proxy_hide_header Referrer-Policy;
        proxy_hide_header X-Content-Type-Options;
        proxy_hide_header X-Frame-Options;
        proxy_hide_header X-XSS-Protection;
    }

    location ~* ^/transmission/?$ {
        return 301 https://$server_name/transmission/web/;
    }
}
```

Start & Optimise:

```console
ln -s /etc/nginx/sites-available/transmission.conf /etc/nginx/sites-enabled/transmission.conf
sudo find -L . -type f ! -iname "*.gz" ! -iname "*.png" ! -iname "*.jpg" ! -iname "*.jpeg" ! -iname "*.gif" ! -iname "*.webp" ! -iname "*.heif" -exec gzip --best -kf "{}" \;
```



