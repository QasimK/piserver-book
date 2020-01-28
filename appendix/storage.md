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



