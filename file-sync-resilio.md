# File Sync App \(with Resilio\)

Resilio is a closed-source-but-free app that does basic file syncing across devices. Its encrypted folders can be used to sync two devices using a third, untrusted intermediary without all devices having to be on at the same time. In our case, we can use the PiServer as the untrusted, always-on device \(especially if the file system on the PiServer does not use full-disk encryption\).

Resource Consumption @ Idle: ~1% CPU, and ~20 MB RAM.

See alternative: NextCloud

\(Alternatives might include syncthing, though this does not support encrypted-only folders.\)

## Security

1. The storage directory for our cloud files is restricted to the service

2. [ ] Secure web interface \(it is HTTP!\)

3. [ ] Harden Systemd

## Install

We install Resilio as a system service \(if more than one person will use it, then we could install it as a user service instead\)

```console
sudo pacmatic -S --needed binutils
git clone https://aur.archlinux.org/rslsync.git
cd rslsync
makepkg -sirc
sudo systemctl enable --now rslsync
```

## Configure

Resilio Sync can be configured entirely through the web interface `http://piserver.local:8888`. How to use this application is out-of-scope for this guide.

* [ ] We may optionally restrict the listening port to `localhost` for security reasons \(e.g. password sniffing\) in  `/etc/rslsync.conf`.

We may optionally configure the storage path. If PiStorage was previously set up, then we can add a new folder to `/mnt/pistorage`

```console
sudo su - root
cd /mnt/pistorage
mkdir resilio
chown rslsync:rslsync resilio
chmod 0750 resilio
```

### Tips

We might have said that how to use this application was out-of-scope, but that does not preclude a few hints...

* To migrate an existing folder into an "encrypted folder", create the new folder and copy the files across. If we want to keep the same folder name, then remove the old folder from the interface and rename it.
* If we do not have a subscription, then, while our separate devices will have separate identities, we can use the same identity name. The device name can be changed later. If we want to change the identity name, then we can unlink it on the device and re-sync the folders.
* The mobile camera backup cannot use encrypted folders.

## Backup

Add the following files to the backup script

```
    /etc/systemd/system/multi-user.target.wants/rslsync.service \
    /var/lib/rslsync \
```

We may also want to backup our cloud files themselves, wherever we have stored them.

## Restore

1. Re-install Resilio Sync.
2. Restore files from backup.

## Conclusion

Resilio Sync is a simple application which we set up on the PiServer to use as an always-on syncing server.

