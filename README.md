# PiServer Book

> This book is largely incomplete.
>
> You can contact me at &lt;...&gt;, but I don't do support. Also take a look at &lt;x&gt; forum for community support.

This book is about building a PiServer - an internet-connected, always-on Raspberry Pi \(or just "Pi"\) in your home.

I started writing this book because I realised my blog was just posts about doing things on the Raspberry Pi, and I had a moment of inspiration from other, free online books such as Game Programming Patterns.

We have the following assumptions, but they are not pre-requisites if you're willing to tweak things as you go along:

* We are at home with a normal computer and also a Raspberry Pi. \(I used the Raspberry Pi 3 Model B while writing this book.\)
* We don't worry about the device being compromised physically **\[1\]**, i.e. if someone picks it up and takes the SD card out.
* We worry about the device being compromised over the network or remotely.
* We are not afraid of the command-line.
* We can port-forward our home router.
* We want to learn new things :\)

As we are making a Linux server more than using device for hardware or electronics work, we can actually use anything in place of the Raspberry Pi - like an old PC, laptop or another ARM device. We use the Raspberry Pi because it is inexpensive, low-electricity consuming, compact, and has good-enough performance.

We'll stick to a headless server, i.e. without a connected monitor, keyboard or mouse, but stuff works either way.

\(We won't worry about Denial of Service attacks from the device itself, e.g. bad users or malware taking up all the disk space or CPU.\)

## Hardware Requirements

* A Raspberry Pi.
* A USB power supply **\[2\]** \(not supplied with the Pi\). This must be at least 2 amps; 2.5 amps is the official recommendation.
* A micro SD card **\[3\]** \(not usually supplied with the Pi\).
* SD card reader for the installation \(e.g. a USB SD card reader\).
* Your main computer, e.g. a desktop or laptop.
* Internet connection for the Raspberry Pi \(ethernet is recommended\).
* Optionally, an extra USB Flash Drive **\[3\]**.
* Optionally, a nice little case.

I personally use a 64GB micro SD card and a single compact 64 GB USB flash drive.

The Raspberry Pi is highly IO constrained - both the ethernet and USB devices share the same USB2 Bus \(480Mbps inc. overhead\). SD cards usually have very poor performance for general purpose computing \(they are optimised for reading and writing large files, rather than a lot of random small ones\). Therefore, I recommend using a high quality micro SD card. TBD: Does the micro SD card share the USB2 Bus?

### Alternatives devices

You may want better compute, storage or networking performance, more RAM, or may already have something else at hand:

* A traditional general purpose computer like a desktop or laptop that you will never turn off.
* Other single-board computers \(SBCs\) like the Orange Pi, Banana Pi, ODROID, Asus Tinker Board, etc.

The initial installation onto these devices will be different, but after that it's all the same.

---

**\[1\]** Some notes on security:

We don't worry about physical security too much as the Raspberry Pi is not well-suited for this. It does not have a TPM or other forms of boot-time protection, so we are limited in how far we can go if the device is powered off \(perhaps you could try to monitor for those events\).

However, it is possible to upgrade the physical security \(i.e. when they can hold the device\) from "someone can copy the data from the attached storage devices without me knowing/just straight up steal the device" to "someone can modify the Pi without me knowing aboutit to steal my encryption password". The latter reduces certain opportunistic attacks.

* [ ] I would and should encrypt the filesystem at least, but did not originally due to performance concerns, which can be tested.
* [ ] It's possible to use a small SSH server with the boot loader that lets you decrypt the OS via SSH \(Drop Bear SSH\).

**\[2\]** Some notes on the power supply:

* You may get log warnings, brown-outs, and data corruption if the power supply is under.
* \(Link Source\). \(Varies between model.\) The CPU uses up to 1.2 amps by itself, and up to a further 1.2 amps in total for the USB 2 devices \(0.5 amps max for each\).
* We can, and do, save a little bit of power [by disabling HDMI, Wi-Fi and bluetooth](https://www.jeffgeerling.com/blogs/jeff-geerling/raspberry-pi-zero-power).
* A UPS may be useful to avoid data corruption in the event of power loss as Raspberry Pi's can be sensitive to this. Pis don't consume much power so a pass-through power bank is sufficient \(note: some "pass-through" chargers do not seamlessly switch to the battery and will cause your Raspberry Pi to reboot which is useless\).

**\[3\]** The following storage configurations for the operating system are possible:

* 256MB micro SD card for booting, with at least one 2GB+ USB Flash Drive - much more effort to install though!
  * You might want to do this to use an old micro SD card or to rely on the flash drive more.
* 2GB+ micro SD card, with whatever size USB Flash Drive for non-OS data storage.



