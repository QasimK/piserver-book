# System Monitoring \(with Netdata\)

Setting up a comprehensive system monitoring tool can be quite involved. Please see the separate guide: System Monitoring \(Netdata\).

## Security

1. By default the web interface accepts all connections, we configure it to listen to localhost. \(While the web interface is read-only, it reveals sensitive information.\)

## Install

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

[health]
    script to execute on alarm = /etc/netdata/alarm-notify.sh
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

We will enable email alerting with:

```
sudo gpasswd -a netdata mail
sudo cp /usr/lib/netdata/plugins.d/alarm-notify.sh /etc/netdata/alarm-notify.sh
```

> To enable email alerting \(assuming you have set up msmtp\), we must modify `notify-alarm.sh` because it is using directories it shouldn't be.
>
> Line 162-164:
>
> * /etc/netdata
> * /usr/lib/netdata/conf.d
> * /var/cache/netdata
>
> Note that \[health\] and copying alarm-notify.sh would also be unneccessary if this was fixed.

Test with: `sudo -u netdata ./alarm-notify.sh test`

### Backup

```
    /etc/systemd/system/multi-user.target.wants/netdata.service \
    /etc/netdata/netdata.conf \
    /etc/netdata/charts.d.conf \
    /etc/netdata/health_alarm_notify.conf \
    /etc/netdata/alarm-notify.sh \
```

### Restore

1. `gpasswd -a netdata mail`

Monitoring netdata:

* Logs can be viewed with s`sudo journalctl -u netdata`
* Memory usage and KSM can be viewed with netdata itself, of course!

Performance:

* It runs with process scheduling policy = idle by default - this is lower than nice=19.

TODO:

* [ ] You can configure Nginx to show the dashboard over-the-internet. [https://docs.netdata.cloud/docs/high-performance-netdata/](https://docs.netdata.cloud/docs/high-performance-netdata/)

  * [ ] Nginx password: [https://www.geekytuts.net/shell-tricks/password-protect-netdata-with-nginx-permission-denied-while-connecting-to-upstream-error/](https://www.geekytuts.net/shell-tricks/password-protect-netdata-with-nginx-permission-denied-while-connecting-to-upstream-error/)

  * [ ] Nginx conf: [https://docs.netdata.cloud/docs/running-behind-nginx/](https://docs.netdata.cloud/docs/running-behind-nginx/)

* [ ] Perf: [https://docs.netdata.cloud/docs/performance/](https://docs.netdata.cloud/docs/performance/)



