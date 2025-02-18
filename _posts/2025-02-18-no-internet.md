---
layout: post
title:  "Block internet access for applications"
categories: [Linux Scripting]
tags: [linux,firewall,app,application,iptables,nftables]
image:
  path: /assets/img/2025-02-18-no-internet.jpg
last_modified_at: 2025-02-18 19:47:00 +0100
---
This tutorial is ment to be an example to block specific applications from accessing the internet. This is accomplished by the use of bash scripting and iptables firewalling.

## Create a new group for iptables

At first add a new group to the system called `no-internet`.
```bash
groupadd no-internet
```

At second add the new group to your user.
```bash
usermod -aG no-internet $USER
```

At last verify that all went correct.
```bash
# check if group exists:
grep no-internet /etc/group

# check if group is added to your user.
sudo groups $USER
```

## Create iptables rule

If not already check if iptables service is enabled and started.

```bash
# Check if the service is running and autostarting
systemctl status iptables

# Start and enable iptables service
systemctl enable iptables --now
```

Create the new iptables rule.
```bash
iptables -I OUTPUT 1 -m owner --gid-owner no-internet -j DROP
```

Verify that the rule is applied.
```bash
iptables -nvL
```

Make the ruleset persistent after restart.
```bash
iptables-save -f /etc/iptables/iptables.rules
```

## Create start script and usage

Best practise is to create the script at a location available in your $PATH variable. So it could be in `/usr/local/bin`.
```bash
#!/usr/bin/bash
sg no-internet "$@"
```
{: file="/usr/local/bin/no-internet"}

You're finished! Now you can your app by running `no-internet firefox` in terminal. It is also possible to just edit the `Exec=`-Section in `*.desktop`-files.
