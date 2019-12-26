# Personal VPN \(with WireGuard\)

We can create a personal VPN to securely gain remote access to your home network.

Maybe also: Device -&gt; Personal VPN \(access home network\) -&gt; VPN provider \(access internet\)?

We use WireGuard because it is easier to configure, more secure, more reliable and more performant, than OpenVPN. However, it is currently in beta, so I guess it's a good thing this guide is a placeholder right now ;\)

\(Alternatives include OpenVPN.\)

## Security

Wireguard has not been audited yet.

TODO: Use preshared keys.

## Installation

```console
sudo pacmatic -S --needed wireguard-tools wireguard-dkms
```

## Server Setup

```
cd /etc/wireguard
wg genkey > privatekey
chmod 0600 privatekey
wg pubkey < privatekey > publickey
chmod 0644 publickey
```



