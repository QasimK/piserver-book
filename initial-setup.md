# Initial Setup

Now we can move the SD card into the Raspberry Pi, connect the power \(there is no on/off switch\), and SSH in as `alarm`.

We'll perform some basic setup below.

> If at any point it is impossible to access your Raspberry Pi, then connect the micro SD card \(containing your operating system\) back to your main computer and edit any files necessary to get it to work again.

## Hostname, Locale

Set a hostname \(piserver\), set your locale.

## Users and Sudo

Create your user. Set up sudo. SSH Key. Delete alarm.

## SSH

Change port to 23, forbid root, etc. etc.

## Firewall - UFW

...

## Simple Outbound Mail

mstmp.

## Extra Security

### umask 027

### usbguard

Since it's headless we don't need those ports.

TBD: What does this protect against exactly?

We cannot just disable the USB controller because that would also disable the ethernet.

