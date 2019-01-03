# Initial Setup \(Arch Linux\)

Now we can move the SD card into the Raspberry Pi, connect the power \(there is no on/off switch\), and SSH in as `alarm`.

We'll perform some basic setup below.

> If at any point it is impossible to access your Raspberry Pi, then connect the micro SD card \(containing your operating system\) back to your main computer and edit any files necessary to get it to work again.

We should set up SSH on our main computer so that we can connect to the piserver simply with `ssh piserver`. This will make things easier.

## Hostname, Locale, Timezone

Set a hostname \(piserver\), set your locale. You may also set a non-UTC timezone.

```
timedatectl set-timezone $(tzselect)
```

## Users and Sudo

We install some basic tools:

* `sudo` is used for security, allowing us to execute root commands without logging in as root.
* `pacmatic` is a simple wrapper around `pacman` which can give warnings when doing system upgrades.
* `vim` is the one true text editor.

```console
sudo pacman -Syu --needed sudo pacmatic vim
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

### Personal Preferences

Configure `~/.pam_environment` for cross-shell environment variables.

```
VISUAL=/usr/bin/vim
```

This is a little more annoying than just using `.bashrc`, but it does let you switch between shells more easily.

## SSH

Change port to 22, forbid root, etc. etc.

## System

### Time

We enable time synchronisation because it is important for many different services. The built-in systemd-timesyncd service is easy to use.

```console
sudo systemctl enable systemd-timesyncd.service
sudo timedatectl set-ntp true
```

### Cache

Pacman will build up an infinite collection of cached system packages in `/etc/cache/pacman/pkg`. We can automatically clean this up when we do a upgrade. To keep the last 7 versions of each package, we create `/etc/pacman.d/hooks/paccache.hook`

```ini
[Trigger]
Operation = Upgrade
Type = Package
Target = * 

[Action]
Description = Removing old cached packages...
When = PostTransaction
Exec = /usr/bin/paccache -rk7
```

It is possible to delete uninstall packages with `paccache -ru0` but I am not sure if it is necessary.

## Firewall - UFW

...

## Simple Outbound Mail

mstmp.

* [ ] Migrate from blog.

## Extra Security

* [ ] Delay after login attempts \(user accounts\).
* [ ] Limit number of processes a user may have.
* [ ] Limit users which may login as root: [https://wiki.archlinux.org/index.php/Security\#Allow\_only\_certain\_users](https://wiki.archlinux.org/index.php/Security#Allow_only_certain_users)
* [ ] Kernel hardening: https://wiki.archlinux.org/index.php/Security\#Kernel\_hardening

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



