# Torrent App \(with Transmission\)

Transmission is a simple, no-frills torrenting application with different GUI, CLI and web interfaces. We will set up a service that works through a VPN that uses the data storage area of the Pi with a secure web interface to manage it. This will allow the peer-to-peer torrent traffic to be more securely segmented from other services that you may run, and only have access to one open port.

\(Alternatives might include rtorrent or deluge.\)

## Security

1. Transmission will run as its own user `transmission` .
2. It will have access to only its own files and a special data directory.
3. It will operate inside a VPN \(using OpenVPN\) that forwards exactly one port.
4. The web interface will be secured with HTTPS and will be accessible over the LAN only.
5. The data storage directory will be accessible to selected users.

## VPN

Something something split-tunneling not going to do it?

## PiStorage/Torrent

mkfs.f2fs needs benchmarking for optimal settings if you want. Otherwise -d lets you use USB storage easily.

## Web Interface

`socat`, like the name suggests, can connect sockets together. A socket could be a TCP socket, a Unix socket, or something else.

We'll create a systemd service that will

## Port Fowarding

The torrent client can seed much more effectively  if you have an open port that allows new, incoming connections from other peers.

## Remote File Access

See alternatives: NAS \(Samba\).

Neither of NFS and Samba are particularly secure \(or the secure variants are a pain to set up\), so we will use SSHFS which will allow us to access the completed files from any \(capable\) device that we can SSH in from.

## Backup

Update the backup file:

```
/var/lib/transmission/config/config.json or something
```

## Conclusion

We set up a secure torrent server

* that ensures our privacy, and
* that provides access to a web UI both on the LAN and through the internet, and
* that is a good citizen that will seed torrents, and
* that allows access to the files remotely, and
* that will backup the configuration files for easy restore.



