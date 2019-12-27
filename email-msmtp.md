# Email \(with msmtp\)

We can configure our PiServer to send emails when systemd units fail. This can also be used to send emails in general.

It is important to note that `msmtp` is very simple. So simple, that without a working internet connection sending emails will simply fail permanently.

TODO: Further explanation can be found [https://qasimk.io/2018/linux-email/](https://qasimk.io/2018/linux-email/).

## Security

* Secret keys in the configuration is secured under root.
* Only users in the `mail` group can send emails.

## Third-Party Service

Residential IPs have very poor reputation amongst email providers, therefore we will use a third-party service to send emails, for example Sendgrid, or Gmail.

\(In this example, we use Sendgrid.\)

## Install & Configuration

```console
pacman -S --needed msmtp msmtp-mta
```

Configure `/etc/mstmprc`:

```
defaults
aliases /etc/aliases
port 587
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
syslog on

account sendgrid
host smtp.sendgrid.net
auth on
user apikey
password ACTUALAPIKEYHERE
auto_from on
maildomain em.example.com

account default : sendgrid
```

## Test

```console
gpasswd -a <YOU> mail
printf "Subject: Hello World\n\Or rather just me.\n" | msmtp default
```

## Backup

```console
    /etc/msmtprc \
    /etc/aliases \
```

## Conclusion

TODO:

