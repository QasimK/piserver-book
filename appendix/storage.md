# Storage & Filesystem

## SMART

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



