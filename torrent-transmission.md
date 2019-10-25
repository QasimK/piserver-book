# Torrent App \(with Transmission\)

Transmission is a simple, no-frills torrenting application with different GUI, CLI and web interfaces. We will set up a service that works through a VPN that uses the data storage area of the Pi with a secure web interface to manage it. This will allow the peer-to-peer torrent traffic to be more securely segmented from other services that you may run, and only have access to one open incoming port.

\(Alternatives might include rtorrent or deluge.\)

## Security

1. Transmission will run as its own user `transmission` .
2. It will have access to only its own files and a special data directory.
3. It will operate inside a VPN \(using OpenVPN\) that forwards exactly one port.
4. The web interface will be secured with HTTPS and will be accessible over the LAN only.
5. The data storage directory will be accessible only to specific users.

## VPN

Something something split-tunneling not going to do it?

To run a single application through the VPN rather than the entire system, we can use network namespaces.

* [ ] Modify /etc/openvpn/client/move-to-netns.sh with vpn -&gt; "$NETNS\_NAME" and `if [ -z ${NETNS_NAME+x} ]; then exit 99; fi`

We will create `/etc/systemd/netns@.service`

```ini
[Unit]
Description=Named network namespace %i
Documentation=https://github.com/systemd/systemd/issues/2741#issuecomment-336736214
StopWhenUnneeded=yes

[Service]
Type=oneshot
RemainAfterExit=yes

# Ask systemd to create a network namespace
PrivateNetwork=yes

# Ask ip netns to create a named network namespace
# (This ensures that things like /var/run/netns are properly setup)
ExecStart=/sbin/ip netns add %i

# Drop the network namespace that ip netns just created
ExecStart=/bin/umount /var/run/netns/%i

# Re-use the same name for the network namespace that systemd put us in
ExecStart=/bin/mount --bind /proc/self/ns/net /var/run/netns/%i

# Clean up the name when we are done with the network namespace
ExecStop=/sbin/ip netns delete %i
```

Then we will layer a new service that moves OpenVPN into that created network namespace `/etc/systemd/netns-openvpn@.service`

```ini
[Unit]
Description=Create the network namespace "%i" routed through OpenVPN
StopWhenUnneeded=yes

[Service]
Type=simple
Environment=NETNS_NAME=%i
ExecStart=/bin/openvpn --cd /etc/openvpn/client/ --config config.ovpn --config override.conf
```

* [ ] TODO: Harden OpenVPN

## Transmission

Edit the transmission service `sudo systemctl edit transmission`

```ini
[Unit]
BindsTo=netns-openvpn@vpn.service
After=netns-openvpn@vpn.service
Before=transmission-socket.service
Wants=transmission-socket.service
JoinsNamespaceOf=netns@vpn.service

[Service]
PrivateNetwork=true
EnvironmentFile=/run/openvpn-port-netns-vpn.env
ExecStart=
ExecStart=/usr/bin/transmission-daemon --foreground --log-error --peerport ${openport}
```

Now start the service.

Note to see logs alter override of transmission with: \(--log-error, --log-info, --log-debug\)

```ini
ExecStart=/usr/bin/transmission-daemon --foreground --log-debug --peerport ${openport}
```

If we look at the logs we might see: [https://falkhusemann.de/blog/2012/07/transmission-utp-and-udp-buffer-optimizations/](https://falkhusemann.de/blog/2012/07/transmission-utp-and-udp-buffer-optimizations/)

Therefore, we should increase these buffer sizes `sudoedit /etc/sysctl.d/90-transmission-buffer.conf`

```ini
net.core.rmem_max = 4194304
net.core.wmem_max = 1048576
```

## PiStorage/Torrent

mkfs.f2fs needs benchmarking for optimal settings if you want. Otherwise -d lets you use USB storage easily.

## Web Interface

`socat`, like the name suggests, can connect sockets together. A socket could be a TCP socket, a Unix socket, or something else.

Hmm, maybe just see Nginx?

* [ ] TODO: Change this to TCP localhost:9091
* [ ] Harden Systemd - Add to Nginx.

```ini
[Unit]
Description=Connect Transmission
BindsTo=transmission.service
After=transmission.service

[Service]
Type=simple
ExecStart=/usr/bin/socat -d UNIX-LISTEN:'/run/transmission.sock,mode=0770,group=http,fork' exec:'ip netns exec vpn "socat STDIO TCP-CONNECT:127.0.0.1:9091,nodelay",nofork,pipes'                                 
Restart=on-failure
```

## Port Fowarding

The torrent client can seed much more effectively  if you have an open port that allows new, incoming connections from other peers.

...

## Remote File Access

See alternatives: NAS \(Samba\).

Neither NFS nor Samba are particularly secure \(or the secure variants are a pain to set up\), so we will use SSHFS which will allow us to access the completed files from any device that we can SSH in from.

We create the mount directory on our normal devices \(not the PiServer\) for our user: `~/Mount/piserver/torrent`. We can mount the directory on the PiServer with

```
sshfs -o auto_unmount,kernel_cache,auto_cache,delay_connect piserver:/mnt/pistorage/torrent ~/Mount/piserver/torrent
```

* `auto_unmount`
* `kernel_cache`
* `auto_cache`
* `delay_connect` Connect when the directory is first opened
* \( `Ciphers=arcfour` was not allowed.\)
* \(`Compression=no` did not help CPU usage nor transfer speed - limited at 100 Mbit/s. Err is that my LAN's fault?\)

Unmount with `fusermount3 -u ~/Mount/piserver/torrent`.

* [ ] Auto-mount with `/etc/fstab`.

## Backup

Update the backup file:

```
    /etc/openvpn/client \
    /usr/local/bin/vpnbox \
    /etc/systemd/system/netns@.service \
    /etc/systemd/system/netns-openvpn@.service \
    /etc/systemd/system/transmission.service.d/override.conf \
    /var/lib/transmission/.config/transmission-daemon/settings.json \
```

## Conclusion

We set up a secure torrent server

* that ensures our privacy, and
* that provides access to a web UI both on the LAN and through the internet, and
* that is a good citizen that will seed torrents, and
* that allows access to the files remotely, and
* that will backup the configuration files for easy restore.



