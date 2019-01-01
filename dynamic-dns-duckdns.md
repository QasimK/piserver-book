# Dynamic DNS \(with DuckDNS\)

Dynamic DNS will let us access our PiServer over the internet even though our home IP address might be changing, as is common with consumer ISPs.

An easy way to find out our external IP address \(i.e. the one that our router has\) is `curl ifconfig.co`.

While any Dynamic DNS provider can be used, we will use DuckDNS as an example because it is free and easy-to-use. The steps outlined in this chapter can be adapted for any other Dynamic DNS provider.

See alternative: Tor.

## Security

1. The file containing the secret token will be accessible only to `root`.

## DNS and Custom Domains

A custom domain is not necessary, because our Dynamic DNS provider will \(usually?\) give us one. For example,  `example.duckdns.org`.

However, if we have one then we can add a `CNAME` record for our nicer domain, e.g. `piserver.example.com` to point to the uglier domain.

We can further chain the `CNAME` records, for example, we might point `cloud.example.com` -&gt; `piserver.example.com`.

## Secret Token

DuckDNS uses a secret token. We will create this inside `/root/secrets/duckdns`:

```ini
TOKEN=<TOKEN>
```

Replace `<TOKEN>` with the token you obtain from the service. We will ensure that no one else can read the token:

```bash
chmod 0750 /root/secrets
chmod 0600 /root/secrets/duckdns
```

Ensure `/root/.tarignore` excludes these secrets with the line:

```
secrets
```

## Service & Auto-update Script

The systemd script \(replacing the placeholder `<DOMAIN>`\) at `/etc/systemd/system/duckdns.service`

```ini
[Unit]
Description=Update the dynamic IP address on DuckDNS
After=network-online.target
Wants=network-online.target
# OnFailure=notify-email@%N.service

[Service]
Type=oneshot
EnvironmentFile=/root/secrets/duckdns
ExecStart=/usr/bin/curl --max-time 30 "https://www.duckdns.org/update?domains=<DOMAIN>&token=${TOKEN}&ip="
SyslogIdentifier=duckdns
```

* This can be executed with `sudo systemctl start duckdns`.
* This can be monitored with `sudo systemctl status duckdns` or `sudo journalctl -u duckdns`.

The above is a systemd service that simply runs the command and exits. We can create a systemd timer to run this oneshot command periodically at `/etc/systemd/system/duckdns.timer`

```ini
[Unit]
Description=Run duckdns.service every minute

[Timer]
OnBootSec=30
OnUnitActiveSec=1min

[Install]
WantedBy=timers.target
```

We'll start the timer 30 seconds after boot to let the system settle, and thereafter every minute after the last execution.

* This can be enabled with `sudo systemctl enable --now duckdns.timer`

## Port-Forwarding & Testing

* We can test the DNS records using `dig example.duckdns.org` and  `dig piserver.example.com`.
* We will almost certainly need to port-forward incoming connections to the PiServer on our router.
* If we are testing SSH, we may struggle to SSH in to our router's public IP address, so we could try using a mobile connection \(e.g. our phone with a WiFi hotspot\).
  * [ ] Why is this \(sometimes?\) a problem?

## Monitoring

If we have set up monitoring on our PiServer, we can un-comment `OnFailure` in the above service file.

## Backup

Add the following lines to the backup script:

```
    /etc/systemd/system/duckdns.service \
    /etc/systemd/system/duckdns.timer \
    /etc/systemd/system/timers.target.wants/duckdns.timer \
```

### Restore

1. Re-create `/root/secrets/duckdns` as described in the Secret Token section.

## Conclusion

We looked at why a Dynamic DNS service is useful for us, and created a systemd service and timer to automatically update our Dynamic DNS service provider with our PiServer's IP address.

