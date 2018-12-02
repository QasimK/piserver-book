# Backup

Backup is the first step where you actually do something that is not just configuration. It is also the most important. After this, you can pick and choose what you want to do.

Backups are important!

...

You can skip this step and come back to it, if you really want...

## Main Computer Backup

There are many ways to backup and many tools that can do this. Rather than configure a particular tool, we will take the opportunity to learn about `tar`.

The following script can be executed on your normal computer.

* [ ] Encrypt .tar.gz file.

```bash
#!/bin/bash
ssh piserver "\
    sudo tar -cvz --preserve-permissions --one-file-system \
    --exclude-backups --exclude-caches-all --exclude-ignore=.tarignore \
    /boot/config.txt \
    /etc/fstab \
    /etc/group \
    /etc/sudoers.d/ \
    /home/ \
    /root/
    " > piserver-backup.tar.gz
```

* `-c`
* `-v`
* `-z`
* `--preservice-permissions`
* `--one-file-system`
* `--exclude-backups`
* `--exclude-caches-all`
* `--exclude-ignore=.tarignore`

## Off-Site Backup

Now that we have the encrypted backup file, we can upload it somewhere. Perhaps upload the file to your Google Drive?

# 



