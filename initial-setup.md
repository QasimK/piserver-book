# Initial Setup \(Arch Linux\)

> If at any point it is impossible to access your Raspberry Pi, then connect the micro SD card \(containing your operating system\) back to your main computer and edit any files necessary to get working again.

We can now insert the SD card into the Raspberry Pi, and connect the power \(there is no on/off switch\).

We should set up SSH on our main computer so that we can connect to the piserver simply with `ssh piserver`. On our main computer we edit `~/.ssh/config`

```
Host piserver
    HostName <PISERVER IP ADDRESS>
```

Substituting in the IP address of the Raspberry Pi. We could:

* Try lots of random IP addresses until we find the right one \(they're usually `192.168.1.x`, and we can `ping` to narrow down the list\),
* Search for machines with `nmap 192.168.1.0/24 -sL | grep -i pi`
* Search for machines with open Port 22 with `sudo nmap -sT --open -p 22 192.168.1.0/24`
* Use the MAC address on the Raspberry Pi obtained from `ip link list` and look at the list of devices on our DHCP server \(i.e. our router\),
* Set a static IP address on the Raspberry Pi with `ip addr add`,
* Set a static IP address for the Raspberry Pi on our DHCP server, or
* Set up [**Service Discovery \(Avahi\)**](/service-discovery-avahi.md) so that we can `ssh piserver.local` which works no matter what IP address the was given to the RPi from the DHCP server.

## Users and Sudo

The Arch Linux ARM OS comes with two users:

* `root` with password `root` \(the super-user\), and
* `alarm` with password `alarm`.

This is very easy to use but ultimately insecure, so we will remove access to both accounts.

We need to SSH into the piserver and switch to the root account.

```console
# On your computer
ssh alarm@piserver

# Now on the Raspberry Pi
su - root
```

We install some basic tools:

* `sudo` is used for security, allowing us to execute root commands without logging in as root.
* `pacmatic` is a simple wrapper around `pacman` which can give warnings when doing system upgrades. `python-html2text` is an optional dependency that formats some news items better.
* `vim` is the one true text editor.
* `htop` is an excellent, intuitive monitoring tool.

```console
pacman-key --init
pacman-key --populate archlinuxarm
pacman -Syu --needed sudo pacmatic python-html2text vim htop
```

We can optionally delete some unnecessary tools \(find them with `pacman -Qent`:

```
pacman -Rs net-tools netctl vi wireless_tools wpa_supplicant
```

We create our \(adminstrative\) user and set its password, substituting `<YOU>` for the username we like the most

```console
useradd --groups wheel --create-home <YOU>
passwd <YOU>
```

We give the adminstrative group `wheel` sudo privileges by properly using `visudo -f /etc/sudoers.d/wheel`

```console
%wheel ALL=(ALL) NOPASSWD:ALL
```

We add our SSH key **from our PC**:

```console
ssh-copy-id <YOU>@piserver
```

Now we should logout, and SSH in as our new user to test everything is working including doing a sudo command.

Then we proceed to remove the default `alarm` user and lock `root`.

```console
sudo userdel --remove alarm
sudo passwd --lock root
```

The only way to login as root is to switch to it with `sudo su - root` from your new administrative user account.

## Hostname, Locale, Keymap, Timezone

We set a hostname, locale, keymap, and, if we don't want to use UTC, our local timezone.

```console
sudo hostnamectl set-hostname piserver

# Set the locale - uncomment the locale:
sudoedit /etc/locale.gen
sudo localectl set-locale en_GB.UTF-8
sudo locale-gen

# Set the keymap - view a list of keymaps with:
# localectl list-keymaps
sudo localectl set-keymap uk

# Optionally, set our local timezone
sudo timedatectl set-timezone $(tzselect)

# Set /etc/hosts
127.0.0.1    localhost
::1          localhost
127.0.1.1    piserver.localdomain piserver
```

Setting the keymap now is helpful if you ever need to plug in a keyboard.

### Personal Preferences \(Completely Optional\)

* [ ] Move into the appendix and expand out.

Configure `~/.pam_environment` for cross-shell environment variables.

```ini
VISUAL=/usr/bin/vim
```

This is a little more annoying than just using `.bashrc`, but it does let us switch between shells more easily.

Install the [fish shell](https://fishshell.com/) and enable it for our user:

```console
sudo pacmatic -S --needed fish
chsh -s /usr/bin/fish $USER
```

## SSH

> This is for OpenSSH. TODO: Dropbear?

We should make SSH more secure with some minor tweaks using `sudoedit /etc/ssh/sshd_config`.

```
PermitRootLogin no
AllowGroups wheel
PasswordAuthentication no
```

> Ensure you used `ssh-copy-id` to copy over your SSH key before enabling `PasswordAuthentication no` or else you will lose SSH access to the RPi.

Then, we can test our changes are valid and reload the service to pick up the new config:

```console
sudo sshd -T
sudo systemctl reload sshd
```

## System

### Time

We enable time synchronisation because it is important for many different services. The built-in `systemd-timesyncd` service is easy to use.

```console
sudo systemctl enable --now systemd-timesyncd.service
sudo timedatectl set-ntp true
```

### Pacman Cache

Pacman will build up an infinite collection of cached system packages in `/etc/cache/pacman/pkg`. This is not useful, and it is likely we have limited disk space on the PiServer. We can remove old cached packages using a _pacman_ hook that runs after we execute certain _pacman_ commands.

Keeping some old versions is useful for the \(very\) rare occasion that you want to downgrade.

Ensure the hooks directory is created `sudo mkdir -p /etc/pacman.d/hooks`.

Install `sudo pacman -S --needed pacman-contrib`.

To remove all versions of an uninstalled package, we `sudoedit /etc/pacman.d/hooks/paccache-remove.hook`

```ini
[Trigger]
Operation = Remove
Type = Package
Target = *

[Action]
Description = Removing package cache for uninstalled packages...
When = PostTransaction
Exec = /usr/bin/paccache -ruk0
```

To keep the last 3 versions of each package we still have installed, we `sudoedit /etc/pacman.d/hooks/paccache-upgrade.hook`

```ini
[Trigger]
Operation = Upgrade
Type = Package
Target = *

[Action]
Description = Removing old cached packages...
When = PostTransaction
Exec = /usr/bin/paccache -rk3
```

### Trim Flash Drives

> **To avoid data loss** ensure trim is supported. The following command gives non-zero DISC-GRAN and DISC-MAX values if it is suppotred.
>
> ```console
> sudo lsblk --discard
> ```

Flash-based systems should be "trimmed" regularly to ensure optimal performance. This process physically clears the flash blocks for deleted files, which allows writes to those blocks to happen faster in the future.

SD cards and SSD drives normally support trim, while USB flash drives normally do not.

It is possible trim when the file is deleted, but this is normally unnecessary. Therefore, we will enable `/usr/lib/systemd/system/fstrim.timer` which trims all supported filesystems in `/etc/fstab` weekly.

```console
sudo systemctl enable --now fstrim.timer
```

> On encrypted file systems, this will leak which areas of the drive are empty.

### SD Card Longevity

Flash NAND storage has limited write durability. This can be worse than we think since even a tiny 1 byte write can cause an entire erase-block to be re-written, which might be 2 MB.

In general, leaving plenty of free space allows wear-levelling to function more effectively and increases the speed of the storage.

We can figure out what is writing to the disk with `sudo iotop -bktoqqq`.

#### Filesystem Options

We can mount the filesystem to reduce metadata writes with `noatime` and `lazytime`. The former stops writing file access times entirely, and the latter batches all file time updates to every minute.

```
/dev/mmcblk0p2  /       ext4    rw,noatime,lazytime     0 0
```

> View existing mounts with their options with `cat /proc/mounts`.

In addition to this, we can add a more dangerous mount option `commit=3600`. This batches up all writes to the storage to commit every hour. Generally, this should not be used if you are storing or modifying any files on the RPi—which is certainly the case for our PiServer.

> Check the validity of `/etc/fstab` with `sudo mount -a`.
>
> We can commit all changes to the filesystem using `sync`.

#### Journaling Options

The journal size for systemd is 10% of the filesystem, capped at 4 GiB. This is almost certainly fine for most situations, but it can be reduced in order to leave more free space for very small microSD cards.

The journal normally syncs to disk every 5 minutes for non-critical messages, this can be increased to batch up writes further. This is not recommended as it is likely to make debugging more difficult.

```console
sudo mkdir -p /etc/systemd/journald.conf.d/
sudoedit /etc/systemd/journald.conf.d/00-journal-sdcard.conf
```

```ini
[Journal]
SystemMaxUse=100M
SyncIntervalSec=1h
```

It is also possible to keep log entries entirely in-memory with  `Storage=volatile`, but this is likely to make debugging more difficult.

#### Other Options

We can keep swap disabled both because it is very slow on an RPi, and also because when it is in-use it causes a lot of writes.

* [ ] Consider setting up a tmpfs for `/var/log`, `/var/cache/samba`.

I would not recommend doing the following suggestion because `/var/tmp` is supposed to survive reboots: mount `/var/tmp` in-memory with `tmpfs /var/tmp tmpfs nodev,nosuid,size=50M 0 0`.

## Boot Time

We can improve the boot time by disabling unnecessary systemd services using `systemd-analyze blame` and `systemd-analyze critical-chain`.

For example, disabling:

* `lvm2-monitor.service`
* `lvm2-lvmetad.service`

## Unused Services

pacman -Rs net-tools netctl wireless\_tools vi wpa\_supplicant

## Misc

* [ ] Fix network issues reported by netdata:
* [ ] Does this save?

```
sudo sysctl -a | grep netdev
sudo sysctl net.core.netdev_budget=3000
sudo sysctl net.core.netdev_budget_usecs=4000
```

## Extra Security

There are further steps that we can take, however they offer increasingly diminishing returns. We do not consider MAC/ACLs/SELinux because they are, apparently, a PITA.

* [ ] Delay after login attempts \(user accounts\).
* [ ] Limit number of processes a user may have.
* [ ] Limit users which may login as root: [https://wiki.archlinux.org/index.php/Security\#Allow\_only\_certain\_users](https://wiki.archlinux.org/index.php/Security#Allow_only_certain_users)
* [ ] Kernel hardening: [https://wiki.archlinux.org/index.php/Security\#Kernel\_hardening](https://wiki.archlinux.org/index.php/Security#Kernel_hardening)

### Firewall - UFW

...

Or [https://firehol.org/](https://firehol.org/)

### umask 027

[https://wiki.archlinux.org/index.php/Umask\#Set\_the\_mask\_value](https://wiki.archlinux.org/index.php/Umask#Set_the_mask_value). However, it might be really inconvenient.

### usbguard

Since it's headless we don't need those ports.

TBD: What does this protect against exactly?

We cannot just disable the USB controller because that would also disable the ethernet.

## Other Tips

### Pacman

A few tips for `/etc/pacman.conf`

```
# Misc options
Color
TotalDownload
```



