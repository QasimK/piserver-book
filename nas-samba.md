# NAS \(with Samba\)

> NFS, a Linux-only alternative to Samba, has poor authentication options. Its MAC/IP-address whitelisting is susceptible to spoofing, and setting up Kerberos requires quite a lot of effort.

Samba can be used anywhere. I don't think it is particularly secure - it must not be used over the internet. \(You could use it with a personal VPN though.\)

Warning: Raspberry Pi IO is **very** slow! Up to and including Model 3B+, the USB and ethernet is capped at 480 Mb/s = 60 MB/s - taking overhead into account \(confirm exact\), it is more like 45 MB/s. That means 20 MB/s coming from the hard drive, and 20 MB/s going out to the network. The ethernet up to and including Model 3B is capped at 100 Mb/s, so an actual max of 12.5 MB/s can be transferred. That's comparable to a real-world Wi-Fi connection \(802.11n/ac\). Model 3B+ and 4 are better.

Warning: Raspberry Pi's cannot power hard drives by themselves, so an external powered USB hub, or externally-powered hard drives should be used.

Alternatives: stuff with a web UI.

