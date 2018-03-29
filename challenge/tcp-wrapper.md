# set tcp wrappers as require

## introduce

To protect the VNC server of the shiyanlou, we need to set up a basic security policy for the TCP Wrappers.

The security requirements for VNC services are as follows:

1. Only allow `192.168.0.0/16` subnet and `local` (127.0.0.1) access to VNC services

2. When other IP addresses access the VNC service, the access information log record to `/var/log/vncdeny.log` , and the access is denied.

3. The desktop of the shiyanlou is connected by VNC service. It uses `Xvnc`. When configuring, we should pay attention to it. It is recommended to switch to the character mode to configure and avoid the influence of desktop interruption


## target 

Well configured `hosts.allow` and `hosts.deny` rules according to security requirements

## knowledge points

- Linux Web service
- TCP_wrappers configuration

## lost ❌
```
#hosts.allow
Xvnc:LOCAL 
Xvnc:192.168.0.0/16

#host.deny
ALL : localhost \
      : spawn (/bin/echo %a from %h attempted to access %d >> \
      /var/log/vncdeny.log) \
      : deny
```
> forget Xvnc

## right answer✔
```
#host.allow
Xvnc: 192.168.0.0/16 LOCAL: allow

#host.deny
Xvnc: ALL : spawn (/bin/echo %a from %h attempted to access %d >> /var/log/vncdeny.log) : deny
```
