# PiServer Book

> This book is largely incomplete.
>
> You can contact me at &lt;...&gt; - I don't do support. Also take a look at &lt;x&gt; forum for community support.

This book is about building a PiServer - an internet-connected, always-on Raspberry Pi in your home.

I started writing this book because I realised my blog was just posts about doing things on the Raspberry Pi, and I had a moment of inspiration from other, free online books such as Game Programming Patterns.

We have the following assumptions, but they are not pre-requisites if you're willing to tweak things as you go along:

* We are at home with a normal computer and also a Raspberry Pi. \(I used the Raspberry Pi 3 Model B while writing this book.\)
* We don't worry about the device being compromised physically **\[1\]**, i.e. if someone picks it up and takes the SD card out.
  * [ ] \(I would and should encrypt the filesystem at least, but did not originally due to performance concerns.\)
* We worry about the device being compromised over the network, a lot.
* We are not afraid of the command-line.
* We can port-forward our home router.
* We want to learn new things :\)

As we are making a Linux server more than using device for hardware or electronics work, we can actually use anything in place of the Raspberry Pi - like an old PC, laptop or another ARM device. We use the Raspberry Pi because it is inexpensive, low-powered \(electricity-wise!\), compact, with good-enough performance.

We'll stick to a headless server, i.e. without a connected monitor, keyboard or mouse, but stuff works either way.

\(We won't worry about Denial of Service attacks from the device itself.\)

**\[1\]** The Raspberry Pi does not have a TPM or other forms of boot-time protection, so we are limited in how far we can go if the device is powered off \(so you could try to monitor for those\).

## Hardware Requirements

* Raspberry Pi.
* A USB power supply \[1\] \(not supplied with the Pi\). This must be at least 2 amps; 2.5 amps is the official recommendation.
* A micro SD card \[2\] \(not supplied with the Pi\).
* SD card reader for the installation \(e.g. a USB SD card reader\).
* Your main computer, e.g. a desktop or laptop.
* Recommended, an ethernet internet connection.
* Optionally, an extra USB Flash Drive \[1\].
* Optionally, a nice little case.

\[2\] The following storage configurations for the operating system are possible.

* 256MB micro SD card, with at least one 2+GB USB Flash Drive - much more effort to install!
* 2+GB micro SD card, with whatever size USB Flash Drive for data storage.

I personally use a 64GB micro SD card and a single compact 64 GB USB flash drive.

The Raspberry Pi is highly IO constrained - both the ethernet and USB devices share the same USB2 Bus \(480Mbps inc. overhead\). I recommend using a high quality micro SD card.

\[1\] Some notes on the power supply:

* You may get log warnings, brown-outs, and data corruption if the power supply is under.
* \(Link Source\). \(Varies between model.\) The CPU uses up to 1.2 amps by itself, and up to a further 1.2 amps in total for the USB 2 devices \(0.5 amps max for each\).
* \(Link Source\). We can save a little bit of power by disabling HDMI, Wi-Fi and bluetooth.



