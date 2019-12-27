# Personal VPN \(with WireGuard\)

We can create a personal VPN to securely gain remote access to your home network.

Maybe also: Device -&gt; Personal VPN \(access home network\) -&gt; VPN provider \(access internet\)?

We use WireGuard because it is easier to configure, more secure, more reliable and more performant, than OpenVPN. However, it is currently in beta, so I guess it's a good thing this guide is a placeholder right now ;\)

\(Alternatives include OpenVPN.\)

## Security

Wireguard has not been audited yet.

TODO: Use preshared keys for improved security?

## Installation

```console
sudo pacmatic -S --needed linux-headers wireguard-tools wireguard-dkms
```

I selected `linux-raspberrypi-headers` when prompted during the installation of `linux-headers`.

We need to translate packets from the Virtual Private Network to/from the Local Area Network. The PiServer will act as a router.

Ensure IPv4 forwarding is enabled:

```
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```

`/etc/sysctl.d/99-sysctl.conf`

```
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding=1
```

Or more limited

sysctl -w net.ipv4.conf.eth0.forwarding=1

sysctl -w net.ipv4.conf.pivpn.forwarding=1

Can also do PreUp, PreDown.

## Port Forwarding

Ensure UDP Port 51820 is forwarded to your PiServer on your home router.

Test this with:

```
sudo pacmatic -S --needed openbsd-netcat
nc -vv -u -l 51820
```

Then on a different machine, try to connect to the PiServer and type anything to send messages back and forth.

```
curl iconfig.co
nc -vv -u <IP-ADDRESS> 51820
```

> Get your IP address with `curl ifconfig.co`

## Server Setup

```
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
chmod 0600 privatekey
chmod 0644 publickey
```

Configure the server \(with no peers\) `/etc/wireguard/pivpn.conf`

```
[Interface]
ListenPort = 51820
PrivateKey = <INSERT FROM ABOVE>
Address = 10.200.200.1/24

# Forward traffic
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

wg-quick up down pivpn

Finally enable with:

```
systemctl enable --now wg-quick@pivpn.service
```

## Client Setup

I strongly recommend using qrencode!

```
wg genkey | tee name.privatekey | wg pubkey > name.publickey
```

Generate client conf.

DNS = 192.168.1.1 - whatever it would be on local network. Note - you can combine ths with Pi-Hole.

Add to device.

```
qrencode -t ansiutf8 < name.conf
```

Testing:

Ping.

Browse website.

Look at logs in Wireguard application.

