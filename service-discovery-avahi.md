# Service Discovery

We can broadcast services like SSH over the _local network_, so that if we forgot what port to SSH in to we can easily find out.

To broadcast the hostname `piserver.local`, and the SSH service on port 22:

```console
pacman -S --needed avahi
cp /usr/share/doc/avahi/ssh.service /etc/avahi/services
systemctl enable --now avahi-daemon.service
```

\(I had to restart dbus.service for whatever reason on my PC.\)

We can check if this worked from another machine using `avahi-browse -alrt`. To do this we must follow the complete set of instructions for the other machine at [https://qasimk.io/2017/local-service-discovery/](https://qasimk.io/2017/local-service-discovery/) which includes setting up the domain name resolution.

