# Pi-Hole

Pi-Hole will allow us to, optionally, secure the DNS, blocks ads, monitor DNS queries, and do all of that for the entire LAN.

Unfortunately it doesn't offer DNS-over-TLS or DNS-over-HTTPS out of the box, so we're not gonna bother with it until it does.

[https://github.com/notasausage/pi-hole-unbound-wireguard](https://github.com/notasausage/pi-hole-unbound-wireguard)

## Steps

We will use Nginx.

```
systemctl disable --now systemd-resolved

# As user not root
yay -S --needed pi-hole-server
```

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



