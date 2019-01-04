# Backup

Backup is the first step where you actually do something that is not just configuration. It is also the most important. After this, you can pick and choose what you want to do.

Backups are important!

You can skip this step and come back to it, if you really want to...

But, seriously, backups are important!

* [ ] Remove /etc/fstab from script below.

## Main Computer Backup

There are many ways to backup and many tools that can do this. Rather than configure a particular tool, we will take the opportunity to learn about `tar`.

The following script can be executed on your normal computer.

```bash
#!/bin/bash

# Get password to encrypt backup archive up-front
# https://stackoverflow.com/a/28393320/197488
printf "Password: "
stty -echo
trap 'stty echo' EXIT
read PASSWORD
stty echo
trap - EXIT
printf "\n"

# Sense check password
echo "Password Length=${#PASSWORD}"
sleep 1

# Create backup file
ssh piserver "\
    sudo tar -cvz --preserve-permissions --xattrs --one-file-system \
    --exclude-backups --exclude-caches-all --exclude-ignore=.tarignore \
    --absolute-names --check-links --totals \
    --label=piserver-$(date --iso-8601=seconds) \
    /boot/config.txt \
    /etc/fstab \
    /etc/group \
    /etc/sudoers.d/ \
    /etc/ssh/sshd_config \
    /etc/pacman.conf \
    /etc/pacman.d/hooks/paccache-remove.hook \
    /etc/pacman.d/hooks/paccache-upgrade.hook \
    /home/ \
    /root/
    " \
    | openssl enc -aes-256-cbc -pbkdf2 -pass pass:"$PASSWORD" -out piserver-backup.tar.gz.enc.1

# Atomic replacement of previous backup (if any)
mv -f piserver-backup.tar.gz.enc.1 piserver-backup.tar.gz.enc
```

* `-c`
* `-v`
* `-z`
* `--preserve-permissions`
* `--xattrs` - include extended attributes \(a filesystem feature\)
* `--one-file-system`
* `--exclude-backups`
* `--exclude-caches-all`
* `--exclude-ignore=.tarignore` - Non-recursive ignore files
* `--absolute-names` - the archive is taken from the root of the filesystem - allow file paths starting with `/`.
* `--sparse` - handle sparse files efficiently
* `--check-links` - verifies hard-links
* `--totals` - print size summary for us to manually sense-check
* `--label=piserver-$(date --iso-8601=seconds)` - labels the archive

As we add applications to the Piserver, we will expand the list of files and directories to backup.

The `.tarignore` file can be used to ignore files within a directory. This is useful for not backing up secret credentials, or for saving space by ignoring temporary files.

For example `/root/.tarignore`

```
.bash_history
.cache
.gnupg
.lesshst
.local
.vim
.viminfo
.wget-hsts
```

Or, `/home/user/.tarignore`

```
.bash_history
.cache
.cargo
.ipython
.lesshst
.local
.python-history
.rnd
.rustup
.ssh
.vim
.virtualenvs
.wget-hsts
tmp
__pycache__
```

Or, `/etc/openvpn/client/.tarignore`

```
auth.txt
```

## Restore

The encrypted file can be decrypted with:

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in piserver-backup.tar.gz.enc -out piserver-backup.tar.gz
```

* [ ] TODO: Restore onto fresh PiServer steps.
* [ ] Careful restore of fstab and group files.

## Off-Site Backup

Now that we have the encrypted backup file, we can upload it somewhere. Perhaps upload the file to your Google Drive?

