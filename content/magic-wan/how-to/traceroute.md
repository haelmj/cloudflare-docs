---
pcx_content_type: how-to
title: Run traceroute
meta:
    description: Learn what settings you need to change to perform a useful `traceroute` to an endpoint behind a Cloudflare Tunnel.
---

# Run `traceroute`

If you have a Magic WAN client connected through [GRE, IPsec](/magic-wan/get-started/configure-tunnels/), [CNI](/network-interconnect/) or [WARP](/magic-wan/tutorials/warp/) and want to perform a `traceroute` to an endpoint behind a [Cloudflare Tunnel](/cloudflare-one/connections/connect-apps/), the following settings must be applied for the command to return useful information.

## Inherited TTL value

On the machine where the `traceroute` client is executed, make sure the tunnel device does not inherit the TTL value of the inner packet. This is the default behavior on Linux and can result in unhelpful `traceroute` results:

```sh
$ sudo traceroute -s 10.1.0.100 -I 10.3.0.100
traceroute to 10.3.0.100 (10.3.0.100), 30 hops max, 60 byte packets
 1  * * *
 2  * * *
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  10.3.0.100 (10.3.0.100)  420.505 ms  420.779 ms  420.776 ms
```

Setting the TTL explicitly returns much better results:

```sh
$ sudo ip link set cf_gre type gre ttl 64
$ sudo traceroute -s 10.1.0.100 -I 10.3.0.100
traceroute to 10.3.0.100 (10.3.0.100), 30 hops max, 60 byte packets
 1  10.0.0.11 (10.0.0.11)  58.947 ms  58.933 ms  58.930 ms
 2  173.245.60.175 (173.245.60.175)  61.138 ms  61.316 ms  61.313 ms
 3  172.68.145.21 (172.68.145.21)  367.448 ms  367.532 ms  367.530 ms
 4  mplat-e2e-vm3.c.magic-transit.internal (10.152.0.20)  370.362 ms  370.440 ms  370.522 ms
 5  10.3.0.100 (10.3.0.100)  370.519 ms  370.541 ms  518.152 ms
 ```

 ## WARP client

Some Linux distributions default to a very strict setting for [reverse path filtering](https://sysctl-explorer.net/net/ipv4/rp_filter/). This strict setting attempts to drop fake traffic as a security measure. Performing a `traceroute` with this setting on can unintentionally drop `traceroute` packets. If you use WARP on Linux, set a less strict policy before attempting to perform a `traceroute`:

```sh
$ sudo sysctl -w net.ipv4.conf.CloudflareWARP.rp_filter=2
net.ipv4.conf.CloudflareWARP.rp_filter = 2
$ sudo traceroute -s 172.16.0.2 -I 10.3.0.100
traceroute to 10.3.0.100 (10.3.0.100), 30 hops max, 60 byte packets
 1  169.254.21.171 (169.254.21.171)  48.887 ms  48.894 ms  48.620 ms
 2  173.245.60.175 (173.245.60.175)  49.403 ms  49.519 ms  49.603 ms
 3  172.68.65.7 (172.68.65.7)  357.499 ms  357.519 ms  357.520 ms
 4  mplat-e2e-vm3.c.magic-transit.internal (10.152.0.20)  360.024 ms  360.086 ms  360.078 ms
 5  10.3.0.100 (10.3.0.100)  360.283 ms  360.297 ms  360.489 ms
 ```