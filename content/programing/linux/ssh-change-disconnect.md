---
title: Change SSH disconnect idle settings
tags:
  - dev
  - linux
  - ssh
---

## On Linux or MacOS

Edit ssh config `nano ~/.ssh/config` or add when not exists.

Add following config properties
```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 8
```

`ServerAliveInterval` - Will send a keepalive message to server every x seconds to stay connected to it
`ServerAliveCountMax` - How many missing responses form server can be before quiet connection. Default value is 3.

If you set `ServerAliveInterval` to 60 and leave `ServerAliveCountMax` as it is, this means the keepalive will only wait for 3 * 60 = 180 seconds =  3 minutes before quiting.