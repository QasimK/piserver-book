# System Monitoring \(with systemd\)

**Prerequisites: **[**Email \(msmtp\)**](/email-msmtp.md)**.**

We can set up _systemd_ to notify us by email if a service fails. _Systemd_ actually does this by starting another service.

## Configuration

We can create the notification service at `/etc/systemd/system/notify-email@.service`

```ini
[Unit]
Description=Send email

[Service]
Type=oneshot
ExecStart=/usr/bin/sh -c 'printf "Subject: [systemd] %i Failed" | /usr/bin/msmtp default'
```

This can easily be tested with `systemctl start notify-email@test`.

This notification method will be used by our other systemd services.

## Backup

We add the files to our backup script:

```
    /etc/systemd/system/notify-email@.service \
```



