# Personal VPN \(with WireGuard\)

We can create a personal VPN to securely gain remote access to your home network.

Maybe also: Device -&gt; Personal VPN \(access home network\) -&gt; VPN provider \(access internet\)?

We use WireGuard because it is easier to configure, more secure, more reliable and more performant, than OpenVPN. However, it is currently in beta, so I guess it's a good thing this guide is a placeholder right now ;\)

\(Alternatives include OpenVPN.\)

There is useful documentation available at: [https://github.com/pirate/wireguard-docs](https://github.com/pirate/wireguard-docs)

## Security

* Wireguard is in beta, and has not been audited.
* Wireguard private keys are secured under `/etc/wireguard`.
* We use pre-shared keys for post-quantum resistance.

## Installation

```console
sudo pacmatic -S --needed linux-headers wireguard-tools wireguard-dkms
```

I selected `linux-raspberrypi-headers` when prompted during the installation of `linux-headers`.

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
>
> Any port can be used, but usually port 51820 is used for Wireguard.

## Server Setup

```console
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
chmod 0600 privatekey
chmod 0644 publickey
```

Configure the server \(with no peers\) `/etc/wireguard/pivpn.conf`

```ini
[Interface]
ListenPort = 51820
PrivateKey = <INSERT FROM ABOVE>
Address = 10.200.200.1/24

# Forward traffic (replace eth0 with internet-interface name)
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostUp = sysctl -w net.ipv4.conf.%i.forwarding=1
PostUp = sysctl -w net.ipv4.conf.eth0.forwarding=1
PreDown = sysctl -w net.ipv4.conf.eth0.forwarding=0
PreDown = sysctl -w net.ipv4.conf.%i.forwarding=0
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

The "forward traffic" configuration translates packets from the Virtual Private Network to/from the Local Area Network - the PiServer will be a router.

To enable/disable the network interface:

```console
wg-quick up pivpn
wg-quick down pivpn
```

Finally, enable the service with:

```console
systemctl enable --now wg-quick@pivpn.service
```

> An alternative to temporary and possible-conflicting sysctl IP packet forwarding is to permanently add to `/etc/sysctl.d/99-pivpn.conf`:
>
> ```ini
> net.ipv4.ip_forward = 1
> net.ipv6.conf.all.forwarding = 1
> ```

## Client Setup \(For Each Client\)

We can create the private keys on the server, and use QR codes to add them to our smartphones.

```console
wg genkey | tee name.privatekey | wg pubkey > name.publickey
wg genpsk > name.psk
```

Generate the client configuration `name.conf`:

```ini
[Interface]
Address = 10.0.0.2/32
PrivateKey = <CONTENTS OF CLIENT name.privatekey HERE>
DNS = 192.168.1.1

[Peer]
PublicKey = <CONTENTS OF SERVER publickey HERE>
PreSharedKey = <CONTENTS OF name.psk HERE>
AllowedIPs = 0.0.0.0/0, ::/0
EndPoint = piserver.example.com:51820
PersistentKeepalive = 60
```

* The DNS is set to the DNS server on the LAN. We could set up a [local DNS resolver](/pi-hole.md). By default, Wireguard prevents DNS leaks.
* We direct all IPv4 and IPv6 internet traffic to the VPN, even if the server cannot send IPv6 traffic over the internet. This prevents IPv6 leaks.
* The endpoint is a [Dynamic DNS](/dynamic-dns-duckdns.md) URL.
* We use `PersistentKeepalive` because the _client_ might be behind a NAT, and this will keep the connection open.

Append the client peer to the server configuration `/etc/wireguard/pivpn.conf`:

```ini
[Peer]
# Name = client-name-here
PublicKey = <CONTENTS OF name.publickey HERE>
PreSharedKey = <CONTENTS OF name.psk HERE>
AllowedIPs = 10.0.0.2/32
```

Now generate a QR code which can be used by the smartphone Wireguard app:

```console
qrencode -t ansiutf8 < name.conf
```

There are several levels of testing that we can do:

* Ping the server, DNS host, and the internet `1.1.1.1`.
* Try `drill google.com` or `curl google.com`.
* Try to browse the internet.
* Look at logs in the Wireguard app.

## Backup

Update the backup file:

```
    /etc/wireguard/ \
    /etc/systemd/system/multi-user.target.wants/wg-quick@pivpn.service \
```

## Conclusion

We set up a VPN to our home LAN:

* that provides access to our home LAN, and
* that is private and secure, and
* that will backup the configuration files for easy restore.



