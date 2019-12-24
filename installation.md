# 

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
>
> **WARNING** -w = destructive read-write test.
>
> List bad blocks: `dumpe2fs -b /dev/sdX1`
>
> Find new bad blocks: `fsck -vcck /dev/sdX1`

### Steps

1. Follow the instructions on the wiki page, using 250 MB for boot, 16 GB for root, and keeping the rest as spare.

### Hardware Optimisation and Issues

#### Raspberry Pi 3

Edit `/boot/config.txt` with some or all of these options.

```ini
# Reduce memory allocation to unused GPU, increasing RAM available to OS
gpu_mem=16

# Disable unused WiFi and Bluetooth hardware to save power
dtoverlay=pi3-disable-wifi
dtoverlay=pi3-disable-bt

# Disable unused HDMI port to save power (undocumented - need source link)
hdmi_blanking=2
# Force normal boots without HDMI cable connected
hdmi_force_hotplug=1
# Disable DVI mode over HDMI
hdmi_drive=2
# Disable overscan for TVs
disable_overscan=1

# Improve the boot time
disable_splash=1
boot_delay=0

# Reduce minimum frequency of processor (to save power?)
arm_freq_min=300
```

> Note that the status of the HDMI port can be checked at /opt/vc/bin/tvservice.

#### Booting up with HDMI plugged in

> This likely affects ARMv6 devices including the Raspberry Pi, Raspberry Pi Zero, and Raspberry Pi Zero W.

Some Raspberry Pi boards including the Raspberry Pi Zero W [will not boot up properly](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=11259) without a HDMI display plugged in. To fix this, add this line to `/boot/config.txt`

```ini
hdmi_force_hotplug=1
```

#### Power

Watch out for `Under-voltage detected!` system logs.

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

### f2fs

f2fs encryption info: [https://www.kernel.org/doc/html/v4.15/filesystems/fscrypt.html](https://www.kernel.org/doc/html/v4.15/filesystems/fscrypt.html)

Per-Directory. Encrypt file + filename. Not file size, timestamps, permissions, extended attributes. Uses fscrypt, kernel-tool.

## Example Installation: ARMv7 Encrypted Root

In this brief example of an installation, we will install **Arch Linux ARMv7** on a **Raspberry Pi 3B+**  with a **headless **installation method. We start up by following [the standard instructions](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3).

1. On your PC:
   ```console
   wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz.sig
   wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
   gpg --verify ArchLinuxARM-rpi-2-latest.tar.gz.sig
   ```
2. Verify the SD Card: `badblocks -wsv /dev/sdX`.

3. Partition the SD Card using `fdisk /dev/sdX`:

   1. 250 MB _boot_ FAT32 primary partition.

   2. 3750 MB _clearroot_ primary partition.

   3. The remaining space will be our encrypted root later, and so should also be partitioned.

4. Create and mount the filesystems

   ```console
   mkfs.vfat -n boot /dev/sdX1
   mkdir boot
   mount /dev/sdX1 boot

   mkfs.ext4 -L clearroot /dev/sdX2
   mkdir root
   mount /dev/sdX2 root
   ```

5. Download and install Arch Linux ARM:  



