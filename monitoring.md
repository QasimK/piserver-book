# Monitoring

> Feel free to skip this page and come back to it later.

Monitoring servers is _usually_ important, but it can be quite complex. This includes:

* Resource usage: is the disk getting full? Is the CPU being overloaded?
* Errors: has any service or script experienced a critical error?
* Security: is my server being attacked?
* Alerting: how are we informed of these events? Text Message? Email?

As it is both important and involved, we note down here that we should do this, while the steps are actually in the following separate guides:

* [**System Monitoring \(systemd\)**](/system-monitoring-systemd.md)
* [**System Monitoring \(Netdata\)**](/system-monitoring-netdata.md)

## Commands

Listening services: `ss -alnrp` or variants of it.



