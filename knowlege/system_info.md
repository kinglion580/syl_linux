- **print system version**
```bash

$ cat /proc/version                                                                              [9:59:01]
Linux version 3.10.0-693.5.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Fri Oct 20 20:32:50 UTC 2017
```

- **print the machine hardware name**
```
$ uname -m                                                                                      [10:03:25]
x86_64
```

- **display all of the above information**
```
$ lsb_release -a                                                                                [10:03:40]
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.4.1708 (Core) 
Release:	7.4.1708
Codename:	Core

```
> `command not found`：install `sudo yum install -y redhat-lsb`
