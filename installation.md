# Installation

We're going to use Arch Linux ARM not because we like to move fast and break things, but because it is a minimal base that will let us make the most of the diminuitive hardware and learn more about Linux, the operating system, and commonly used programs.

## Initial Install

bla bla bla.

This is your opportunity to encrypt the root partition \(excluding the boot drive\) or use f2fs with the root partition. I did neither at this point.

Encryption is tricky because you'll need to physically enter the password if you reboot.

**Encryption after SSH? **Worth looking into:

* [http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/](http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/)
* [https://security.stackexchange.com/q/46548](https://security.stackexchange.com/q/46548)
* [https://esther.codes/post-cryptsetup\_raspberry/](https://esther.codes/post-cryptsetup_raspberry/)
* https://docs.kali.org/kali-dojo/04-raspberry-pi-with-luks-disk-encryption

**Create an image for easy future use... \(after initial setup... hmm\)**

> Note: `badblocks -wsv /dev/...` can be used to check each storage device you are using is okay.

### Hardware Optimisation

#### Raspberry Pi 3

Edit `/boot/config.txt`

```
# Reduce memory allocation to unused GPU, increasing RAM available to OS
gpu_mem=16

# Disable unused WiFi and Bluetooth hardware to save power
dtoverlay=pi3-disable-wifi
dtoverlay=pi3-disable-bt

# Disable unused HDMI port to save power (undocumented - need source link)
hdmi_blanking=2

# Reduce minimum frequency of processor (to save power?)
arm_freq_min=300
```

> Note that the status of the HDMI port can be checked at /opt/vc/bin/tvservice.

## File System Variations

The `/boot` partition is fairly inflexible \(confirm encryption?\), it must be FAT32. It is possible to boot over the network, or from a USB flash drive.

The root `/` partition is more flexible, and you can install with other file systems from the get-go, e.g. f2fs or btrfs. Edit cmdline.txt.

It is easy to use RAID, LUKS, LVM and whatever file system you want on pure data devices.

It is much more tricky to install if you want to do those things on your main root `/` partition.

### RAID + LUKS + LVM

Could be useful if you have externally-powered USB storage devices.

Note LVM can actually raid, but the tooling is more opaque and there is less community support - `lvmraid`.

### Btrfs

Well we're using Arch, so why not a slightly dodgy filesystem? It has cool features like copy-on-write, snapshots, fast backups, and... RAID5 corrupting your data.

[https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypted-Btrfs-Root](https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypted-Btrfs-Root)

