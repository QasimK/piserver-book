# Backup

Backup is the first step where you actually do something that is not just configuration. It is also the most important. After this, you can pick and choose what you want to do.

Backups are important!

...

You can skip this step and come back to it, if you really want...

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

ssh piserver "\
    sudo tar -cvz --preserve-permissions --xattrs --one-file-system \
    --exclude-backups --exclude-caches-all --exclude-ignore=.tarignore \
    --absolute-names \
    /boot/config.txt \
    /etc/fstab \
    /etc/group \
    /etc/sudoers.d/ \
    /home/ \
    /root/
    " \
    | openssl enc -aes-256-cbc -pbkdf2 -pass pass:"$PASSWORD" -out piserver-backup.tar.gz.openssl
```

* `-c`
* `-v`
* `-z`
* `--preserve-permissions`
* `--xattrs` 
* `--one-file-system`
* `--exclude-backups`
* `--exclude-caches-all`
* `--exclude-ignore=.tarignore`
* `--absolute-names` - the archive is taken from the root of the filesystem - it is absolute.

As we add applications to the Piserver, we will expand the list of files and directories to backup.

The `.tarignore` file can be used to ignore files within a directory. This is useful for not backing up secret credentials, or for saving space by ignoring temporary files.

For example `/root/.tarignore`

```
secrets
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
openssl enc -d -aes-256-cbc -pbkdf2 -in piserver-backup.tar.gz.openssl -out piserver-backup.tar.gz
```

* [ ] TODO: Restore onto fresh PiServer steps.

## Off-Site Backup

Now that we have the encrypted backup file, we can upload it somewhere. Perhaps upload the file to your Google Drive?

# 



