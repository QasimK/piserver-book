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
    /etc/msmtprc \
    /etc/aliases \
    /etc/systemd/system/notify-email@.service \
```

## System Monitoring

> The /etc/netdata/edit-config command is incorrect. Modify it with `sudoedit edit-config`
>
> Line 12: \[...\] `NETDATA_USER_CONFIG_DIR="/etc/netdata"`
>
> Line 13: \[...\] `NETDATA_STOCK_CONFIG_DIR="/usr/lib/netdata/conf.d"`

We will use netdata.

```
sudo pacmatic -S --needed netdata
sudo systemctl enable --now netdata
```

We will create the main configuration file at `/etc/netdata/netdata.conf`

```
[global]
    # A data point every 10 seconds for 24 hours
    history = 8640
    update every = 10
    error log = syslog
    access log = none

[web]
    bind to = localhost
```

We will also modify the charts configuration file  `sudo -u netdata /etc/netdata/edit-config charts.d.conf`; uncomment the following line at the end:

```
sensors=force
```

To save memory we will enable Kernel Same-page Merging \(KSM\), i.e. RAM deduplication. We run it every 5 seconds \(the default is every 20 ms\) because netdata collects every 10 seconds.

* [ ] TODO: Run the following on boot, 

```
echo 1 | sudo tee /sys/kernel/mm/ksm/run
echo 10000 | sudo tee /sys/kernel/mm/ksm/sleep_millisecs
```

### Backup

```
    /etc/systemd/system/multi-user.target.wants/netdata.service \
    /etc/netdata/netdata.conf \
    /etc/netdata/charts.d.conf \
```

Monitoring netdata:

* Logs can be viewed with s`sudo journalctl -u netdata`
* Memory usage and KSM can be viewed with netdata itself, of course!

Performance:

* It runs with process scheduling policy = idle by default - this is lower than nice=19.

* [ ] It is accessible over the lan: `piserver.local:19999`
* [ ] You can configure Nginx to show the dashboard over-the-internet. [https://docs.netdata.cloud/docs/high-performance-netdata/](https://docs.netdata.cloud/docs/high-performance-netdata/)
* [ ] Email notifications
* [ ] Raspberry Pi sensor [https://docs.netdata.cloud/docs/netdata-for-iot/](https://docs.netdata.cloud/docs/netdata-for-iot/)
* [ ] More charts: [https://docs.netdata.cloud/docs/add-more-charts-to-netdata/](https://docs.netdata.cloud/docs/add-more-charts-to-netdata/)
* [ ] Perf: [https://docs.netdata.cloud/docs/performance/](https://docs.netdata.cloud/docs/performance/)



