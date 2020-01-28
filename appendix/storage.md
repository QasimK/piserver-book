# Storage & Filesystem

## SMART

> These are all safe to run without risk of data loss.
>
> Smart Data can be reset \(buyer beware\)

Install:

```
sudo pacmatic -S --needed smartmontools
```

List all information:

```
sudo smart --all /dev/sdx
```

Run a test \(see above for tests available\):

```
sudo smartctl -t long /dev/sdx
```

Check if test is finished:

```
sudo smartctl -l selftest /dev/sdx
```

Check results of test:

```
sudo smartctl -H /dev/sdx
```

## Badblocks

Run **destructive** read-write test of entire drive:

```
sudo badblocks -wsv -b 4096 -c 65536 /dev/sdx
```

> This runs 4 test patterns and then terminates. Block size 4k matches physical block size. Number of blocks = 256 MB at time.

## Performance Test

## Bit Rot

Parchive??

cryptsetup --integrityX  --sector-size 4096 --label "" --allow-discards \(open only\)

X = hmac-sha256?--integrity hmac-sha512 and --cipher chacha20-random --integrity poly1305

That detects failures. Use mdadm scrub to be able to fix them.

\[0\]: [https://securitypitfalls.wordpress.com/2018/05/08/raid-doesnt-work/](https://securitypitfalls.wordpress.com/2018/05/08/raid-doesnt-work/)

\[1\]: [https://archive.fosdem.org/2018/schedule/event/cryptsetup/](https://archive.fosdem.org/2018/schedule/event/cryptsetup/)

Levels:

\[mdadm - safe disk\]

\[RAID dev1\] \[RAID dev2\]

\[crypt-integrity-AEAD\] \[crypt-integrity-AEAD\]

Not the best idea to use the same disk - performance and disk-failure concerns.

## Encyption

Linux fscrypt - fs file encryption could be useful. Once decrypted \(key added to kernal keyring\), it uses usual unix file permissions to protect access.

Otherwise dm-crypt.

## ext4

-O 64bit,extent,metadata\_csum,discard

-m 0 - for non-root drives, no performance impact unless nearly full and running for long time to cause fragmentation

-T largefile - 1MiB files average limit for inodes \(for backups\).

noatime,lazytime

