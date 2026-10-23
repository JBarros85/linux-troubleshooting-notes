# Logrotate failing because of a duplicate cloud-init configuration on Ubuntu 26.04

## Problem

The daily `logrotate.service` was failing on an Ubuntu 26.04 system.

The timer itself remained active, but every execution of the service returned an error.

Initial inspection:

```bash
systemctl --failed
```

showed:

```text
logrotate.service loaded failed failed Rotate log files
```

## Environment

```text
Ubuntu 26.04.1 LTS
systemd
logrotate
cloud-init 26.1
cloud-init-base 26.1
```

## Diagnosis

The service status showed:

```text
error: cloud-init-base:1 duplicate log entry for /var/log/cloud-init.log
error: found error in file cloud-init-base, skipping
```

The same failure had occurred repeatedly during previous daily executions.

The timer itself was healthy:

```bash
systemctl status logrotate.timer
```

but the service it triggered was returning:

```text
Result=exit-code
ExecMainStatus=1
```

## Configuration test

The complete logrotate configuration was checked without modifying any logs:

```bash
sudo logrotate --debug /etc/logrotate.conf
```

The test confirmed a duplicate definition involving:

```text
/etc/logrotate.d/cloud-init
/etc/logrotate.d/cloud-init-base
```

## Comparing the files

Both files contained the same rule:

```text
/var/log/cloud-init*.log
{
    su root root
    missingok
    nocreate
    notifempty
    rotate 6
    compress
    delaycompress
    size 1M
}
```

Package ownership was checked with:

```bash
dpkg -S /etc/logrotate.d/cloud-init
dpkg -S /etc/logrotate.d/cloud-init-base
```

The files belonged to:

```text
cloud-init
cloud-init-base
```

Further inspection of the package metadata showed:

```bash
dpkg-query -W -f='${Conffiles}\n' cloud-init
dpkg-query -W -f='${Conffiles}\n' cloud-init-base
```

The important result was:

```text
/etc/logrotate.d/cloud-init       obsolete
/etc/logrotate.d/cloud-init-base  current
```

This confirmed that the old `cloud-init` logrotate configuration was obsolete but was still present on disk.

Because both files matched the same log path, logrotate detected a duplicate entry.

## Solution

The obsolete file was moved out of `/etc/logrotate.d` instead of being immediately deleted:

```bash
sudo mv /etc/logrotate.d/cloud-init \
  /root/cloud-init.logrotate.obsolete
```

The active configuration was then:

```text
/etc/logrotate.d/cloud-init-base
```

## Verification

The configuration was tested again:

```bash
sudo logrotate --debug /etc/logrotate.conf
```

No duplicate entry was reported.

The service was then executed normally:

```bash
sudo systemctl daemon-reload
sudo systemctl start logrotate.service
```

The result was checked with:

```bash
systemctl show logrotate.service \
  -p Result \
  -p ExecMainStatus \
  -p ExecMainCode
```

Successful result:

```text
Result=success
ExecMainStatus=0
```

The journal also confirmed normal completion:

```text
logrotate.service: Deactivated successfully.
Finished logrotate.service - Rotate log files.
```

## Why `inactive` is normal afterwards

`logrotate.service` is started periodically by `logrotate.timer`.

After completing its work successfully, the service does not need to remain continuously running.

Therefore an inactive state after a successful execution is expected.

The important indicators are:

```text
Result=success
ExecMainStatus=0
```

## Useful diagnostic commands

```bash
systemctl status logrotate.service --no-pager -l

journalctl -u logrotate.service -n 80 --no-pager

systemctl status logrotate.timer --no-pager

sudo logrotate --debug /etc/logrotate.conf

dpkg -S /etc/logrotate.d/cloud-init

dpkg-query -W -f='${Conffiles}\n' cloud-init
```

## Lesson learned

When a configuration references the same log file more than once, logrotate can reject the configuration and cause its scheduled service to fail.

In this case, the important clue was not simply that two files existed.

The decisive evidence was that:

```text
/etc/logrotate.d/cloud-init
```

was explicitly marked by `dpkg` as an **obsolete conffile**, while:

```text
/etc/logrotate.d/cloud-init-base
```

was the current configuration.

Checking package ownership and package metadata before removing system configuration avoided guessing which file should remain.
