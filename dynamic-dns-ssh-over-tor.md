# Dynamic DNS \(with Tor\)

Tor can be used as a Dynamic DNS provider by registering any type of service in the Tor Hidden Service Directory.

Compared to normal Dynamic DNS services, this has the advantages of not requiring port-forwarding on the router, and hiding your public IP address, and the disadvantages of requiring Tor to be installed on both the server and the client, and reduced performance.

\(This doesn't necessarily hide your identity as the SSH service's public key fingerprint is identical on the Tor service and on the SSH service via your LAN IP address, or via your router's public IP address if you have configured port-forwarding.\)

See alternative: DuckDNS.

## Security

X.

## Overview

This works by...

[https://medium.com/@tzhenghao/how-to-ssh-over-tor-onion-service-c6d06194147](https://medium.com/@tzhenghao/how-to-ssh-over-tor-onion-service-c6d06194147)

