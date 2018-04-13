# Installation

We're going to use Arch Linux ARM not because we like to move fast and break things, but because it is a minimal base that will let us make the most of the diminuitive hardware.

## Initial Install

bla bla bla.

This is your opportunity to encrypt the root partition \(excluding the boot drive\) or use f2fs with the root partition. I did neither at this point.

Encryption is tricky because you'll need to physically enter the password if you reboot.

**Encryption after SSH? **Worth looking into:

* [http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/](http://blog.nguyenvq.com/blog/2011/09/13/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu/)
* [https://security.stackexchange.com/q/46548](https://security.stackexchange.com/q/46548)
* https://esther.codes/post-cryptsetup\_raspberry/

**Create an image for easy future use... \(after initial setup... hmm\)**

### Hardware Optimisation

Edit /boot/config.txt.

## File System Variations

The `/boot` partition is fairly inflexible \(confirm encryption?\), it must be FAT32.

The root `/` partition is more flexible, and you can install with other file systems from the get-go, e.g. f2fs or btrfs. Edit cmdline.txt.

It is easy to use RAID, LUKS, LVM and whatever file system you want on pure data devices.

It is much more tricky to install if you want to do those things on your main root `/` partition.

### RAID + LUKS + LVM

Could be useful if you have externally-powered USB storage devices.

Note LVM can actually raid, but the tooling is more opaque and there is less community support - `lvmraid`.

### Btrfs

Well we're using Arch, so why not a slightly dodgy filesystem? It has cool features like copy-on-write, snapshots, fast backups, and... RAID5 corrupting your data.

