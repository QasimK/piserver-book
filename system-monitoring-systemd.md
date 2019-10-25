## Email on systemd Service Failure

**Prerequisites: Email \(msmtp\)**

We can set up _systemd_ to notify us by email if it fails. _Systemd_ actually does this by starting another service.

* [ ] Add instructions on setting up server emailing from: [http://qasimk.io/2018/linux-email/](http://qasimk.io/2018/linux-email/)
* [ ] Uses GPG to hide the credentials in `/etc/msmtprc`

We can create the service at `/etc/systemd/system/notify-email@.service`

```ini
[Unit]
Description=Send email

[Service]
Type=oneshot
ExecStart=/usr/bin/sh -c 'printf "Subject: [systemd] %i Failed" | /usr/bin/msmtp default'
```

This can easily be tested with `sudo systemctl start notify-email@test`.

This notification method can be used by our other systemd services.

## Backup

We add the files to our backup script:

```
    /etc/msmtprc \
    /etc/aliases \
    /etc/systemd/system/notify-email@.service \
```



