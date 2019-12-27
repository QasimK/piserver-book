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

## Server Setup

```
cd /etc/wireguard
wg genkey | tee privatekey | wg pubkey > publickey
chmod 0600 privatekey
chmod 0644 publickey
```





