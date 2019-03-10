# Installation

We're going to use Arch Linux ARM not because we like to move fast and break things, but because it is a minimal base that will let us make the most of the diminuitive hardware and learn more about Linux, the operating system, and commonly used programs.

## Initial Install

> Raspberry Pi versions 1 - 3B+ \(all of them, circa 2018\) does not support hardware encryption, meaning full disk encryption will use significant CPU % and decrease disk performance.
>
> * [ ] Test.

The initial installation is carried out on our computer following the correct variant of instructions:

* [Raspberry Pi](https://archlinuxarm.org/platforms/armv6/raspberry-pi)
* [Raspberry Pi 2](https://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2)
* [Raspberry Pi 3](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3) \(or, the ARMv7 instructions for the Raspberry Pi 2 may also be followed\)

See [_File System Variations_](#file-system-variations). It's easier to vary this now rather than later.

bla bla bla.

This is your opportunity to encrypt the root partition \(excluding the boot drive\) or use f2fs with the root partition. I did neither at this point.

Encryption is tricky because you'll need to physically enter the password if you reboot.

**Encryption after SSH? **Worth looking into:

* [https://gist.github.com/gea0/4fc2be0cb7a74d0e7cc4322aed710d38](https://gist.github.com/gea0/4fc2be0cb7a74d0e7cc4322aed710d38)
* [http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/](http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/)
* [https://security.stackexchange.com/q/46548](https://security.stackexchange.com/q/46548)
* [https://esther.codes/post-cryptsetup\_raspberry/](https://esther.codes/post-cryptsetup_raspberry/)
* [https://docs.kali.org/kali-dojo/04-raspberry-pi-with-luks-disk-encryption](https://docs.kali.org/kali-dojo/04-raspberry-pi-with-luks-disk-encryption)
* [https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypt-Root-Partition-Tutorial](https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypt-Root-Partition-Tutorial)

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

There is a lot of flexibility in how we configure the partitions.

The `/boot` partition is fairly inflexible \(confirm encryption?\), it must be FAT32. It is possible to boot over the network, or from a USB flash drive.

The root `/` partition is more flexible, and you can install with other file systems from the get-go, e.g. f2fs or btrfs. Edit cmdline.txt.

It is easy to use RAID, LUKS, LVM and whatever file system you want on pure data devices.

It is much more tricky to install if you want to do those things on your main root `/` partition.

We will probably use 3-4 GB on `/` for programs and config files, and all data will be on a separate partition. I would recommend 4x this for  `/` , i.e. 16GB, to leave room for future growth, and significant spare space to extend the longevity of this part of the SD card.

### RAID + LUKS + LVM

Could be useful if you have externally-powered USB storage devices.

Note LVM can actually raid, but the tooling is more opaque and there is less community support - `lvmraid`.

### Btrfs

Well we're using Arch, so why not a slightly dodgy filesystem? It has cool features like copy-on-write, snapshots, fast backups, and... RAID5 corrupting your data.

[https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypted-Btrfs-Root](https://github.com/NicoHood/NicoHood.github.io/wiki/Raspberry-Pi-Encrypted-Btrfs-Root)

