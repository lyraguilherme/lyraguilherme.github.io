---
title: "Resolving SSH Key Exchange Errors"
date: 2024-09-15T16:00:00-03:00
draft: false
cover:
    image: /resolving-ssh-key-exchange-errors/key-exchange.png
    alt: 'Resolving SSH Key Exchange Errors'
    caption: 'Resolving SSH Key Exchange Errors'
---

In this post, I’ll walk you through a few simple steps to resolve SSH Key Exchange and ensure smooth SSH connections.

While working on a network automation task, I encountered an error while trying to establish connections to some legacy devices: 
`Unable to negotiate with x.x.x.x port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1, diffie-hellman-group14-sha1.`

The error message indicates that the only available key exchange methods are **diffie-hellman-group-exchange-sha1** and **diffie-hellman-group14-sha1**. To address this, simply update your `/etc/ssh/ssh_config` file by adding the following lines at the end:

```
HostKeyAlgorithms +ssh-rsa,ssh-dss
KexAlgorithms +diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
```

Make sure to enable the required key exchange methods for your use case and save the `ssh_config` file before attemping a new connection.

By following these steps, you should be able to resolve the key exchange issues and maintain stable SSH connections with legacy devices. However, **keep in mind that you're enabling deprecated and weaker cryptographic algorithms**.