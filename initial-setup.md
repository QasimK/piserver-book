# Initial Setup

Now we can move the SD card into the Raspberry Pi, connect the power \(there is no on/off switch\), and SSH in as alarm.

We'll perform some basic setup.

## Hostname, Locale

Set a hostname \(piserver\), set your locale.

## Users and Sudo

Create your user. Set up sudo. SSH Key. Delete alarm.

## SSH

Change port to 23, forbid root, etc. etc.

Firewall

## Firewall - UFW

## Simple Outbound Mail

mstmp.

## Extra Security

### umask 027

### usbguard

Since it's headless we don't need those ports.

If we disabled the USB controller, we would also disable the ethernet.

