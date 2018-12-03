# Monitoring

> Feel free to skip this page and come back to it later.

Monitoring servers is _usually_ important. This includes:

* Resource usage: is the disk getting full? Is the CPU being overloaded?
* Errors: has any service or script experienced a critical error?
* Security: is my server being attacked?

Simple: [https://my-netdata.io/](https://my-netdata.io/) Not really historical past a few hours. Difficult to zoom graphs.

It's important, hence why this page is empty ;\)

## Email on Service Failure

We can set up systemd to notify us by email if it fails. Systemd actually does this by starting another service.

* [ ] Add instructions on setting up server emailing from: [http://qasimk.io/2018/linux-email/](http://qasimk.io/2018/linux-email/)
* [ ] Uses GPG to hide the credentials in `/etc/msmtprc`

We can create the service at `/etc/systemd/system/notify-email@.service`

```
[Unit]
Description=Send email

[Service]
Type=oneshot
ExecStart=/usr/bin/sh -c 'printf "Subject: [systemd] %i Failed" | /usr/bin/msmtp default'
```

This can easily be tested with `sudo systemctl start notify-email@test`.

This notification method can be used by our other systemd services.

### Backup

We add the files to our backup script:

```
    /etc/aliases \
    /etc/systemd/system/notify-email@.service \
```

> We do not add `/etc/msmtprc` because it currently contains our credentials



