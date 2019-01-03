# Web Server \(with Nginx\)

A web server allows us to securely host web applications over the internet, or on the local LAN.

We can use Nginx to serve other applications that we have set up using TLS encryption over the local network and/or over the internet on our domain. In addition, we can set up basic username/password authentication for any web services without authentication \(e.g. [**System Monitoring \(Netdata\)**](/system-monitoring-netdata.md)\).

\(Alternatives might include: Apache, Lighttpd, or Caddy.\)

## LAN

LAN applications will be served at `piserver.local`. TLS encryption will be done with self-signed certificate which must be installed on your devices.

## Internet

WWW applications will be served at `<DOMAIN>`. TLS encryption will be done using [Letsencrypt](https://letsencrypt.org/) which provides free certificates.

