# Initial Setup \(Arch Linux\)

> If at any point it is impossible to access your Raspberry Pi, then connect the micro SD card \(containing your operating system\) back to your main computer and edit any files necessary to get it to work again.

We can now insert the SD card into the Raspberry Pi, and connect the power \(there is no on/off switch\).

We should set up SSH on our main computer so that we can connect to the piserver simply with `ssh piserver`. On our main computer we edit `~/.ssh/config`

```
Host piserver
    HostName <PISERVER IP ADDRESS>
```

Substituting in the IP address of the Raspberry Pi. We could:

* Try lots of random IP addresses until we find the right one \(they're usually `192.168.1.x`, and we can `ping` to narrow down the list\),
* Use the MAC address on the Raspberry Pi obtained from `ip link list` and look at the list of devices on our DHCP server \(i.e. our router\),
* Set a static IP address on the Raspberry Pi,
* Set a static IP address for the Raspberry Pi on our DHCP server,
* Set up [**Service Discovery \(Avahi\)**](/service-discovery-avahi.md) so that we can set `HostName piserver.local` which works no matter what IP address the router has.

## Users and Sudo

The system comes with two users:

* `root` with password `root` \(the super-user\), and
* `alarm` with password `alarm`.

This is insecure, so we will remove access to both accounts.

We need to SSH into the piserver and switch to the root account.

```console
# On your computer
ssh alarm@piserver

# Now on the Raspberry Pi
su - root
```

We install some basic tools:

* `sudo` is used for security, allowing us to execute root commands without logging in as root.
* `pacmatic` is a simple wrapper around `pacman` which can give warnings when doing system upgrades.
* `vim` is the one true text editor.

```console
pacman -Syu --needed sudo pacmatic vim
```

We create our adminstrative user, substiting `<YOU>` for the username we like the most

```console
useradd --groups wheel --create-home <YOU>
```

We give the adminstrative group `wheel` sudo privileges by properly using `visudo -f /etc/sudoers.d/wheel`

```console
%wheel ALL=(ALL) NOPASSWD:ALL
```

We add our SSH key

```console
ssh-copy-id
```

Now we should logout, and SSH in as our new user to test everything is working.

Then we proced to remove the default `alarm` user and lock `root`.

```console
sudo userdel --remove alarm
sudo passwd --lock root
```

The only way to login as root is to switch to it with `sudo su - root`.

## Hostname, Locale, Timezone

We set a hostname, and our locale. We may also set a non-UTC timezone if we like that kind of thing for our servers.

```
vim /etc/locale.gen
locale-gen
localectl set-locale <LOCALE>

hostnamectl set-hostname piserver

timedatectl set-timezone $(tzselect)
```

* [ ] [https://wiki.archlinux.org/index.php/Installation\_guide\#Localization](https://wiki.archlinux.org/index.php/Installation_guide#Localization)
* [ ] Font & Keymap
* [ ] Does /etc/hosts need to be updated, or does hostnamectl do it?

### Personal Preferences

* [ ] Move into the appendix and expand out.

Configure `~/.pam_environment` for cross-shell environment variables.

```
VISUAL=/usr/bin/vim
```

This is a little more annoying than just using `.bashrc`, but it does let us switch between shells more easily.

Install the fish shell :\)

## SSH

Change port to 22, forbid root, etc. etc.

## System

### Time

We enable time synchronisation because it is important for many different services. The built-in systemd-timesyncd service is easy to use.

```console
sudo systemctl enable systemd-timesyncd.service
sudo timedatectl set-ntp true
```

### Pacman Cache

Pacman will build up an infinite collection of cached system packages in `/etc/cache/pacman/pkg`. This is not useful, and it is likely we have limited disk space on the PiServer. We can remove old cached packages using a _pacman_ hook that runs after we execute certain _pacman_ commands.

Keeping some old versions is useful for the \(very\) rare occasion that you want to downgrade.

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

Flash-based systems should be "trimmed" regularly to ensure optimal performance. This process physically clears the flash blocks for deleted files, which allows writes to those blocks to happen faster in the future.

SD cards and SSD drives normally support trim, while USB flash drives normally do not. The output of `mkfs.f2fs`  reveals whether trim is supported.

* [ ] In installation? What is the output?

It is possible trim when the file is deleted, but this is normally unnecessary. Therefore, we will enable `/usr/lib/systemd/system/fstrim.timer` which trims all supported filesystems in `/etc/fstab` weekly.

```console
sudo systemctl enable --now fstrim.timer
```

> On encrypted file systems, this will leak which areas of the drive are empty.

## Simple Outbound Mail

mstmp.

* [ ] Migrate from blog. Move to "Things we can do".

## Extra Security

There are further steps that we can take, however they offer increasingly diminishing returns. We do not consider MAC/ACLs/SELinux because they are, apparently, a PITA.

* [ ] Delay after login attempts \(user accounts\).
* [ ] Limit number of processes a user may have.
* [ ] Limit users which may login as root: [https://wiki.archlinux.org/index.php/Security\#Allow\_only\_certain\_users](https://wiki.archlinux.org/index.php/Security#Allow_only_certain_users)
* [ ] Kernel hardening: [https://wiki.archlinux.org/index.php/Security\#Kernel\_hardening](https://wiki.archlinux.org/index.php/Security#Kernel_hardening)

### Firewall - UFW

...

Or https://firehol.org/

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



