In addition to the access points, we use a route reflector to build our control plane. It is possible to set up the system without a route reflector, if you are interested in such a setup, just drop us a message. A route reflector can be any kind of machine where frr can be installed. It just needs a L3 connection to the access points. We use an Ubuntu server host.

#### FRR

1. Install `frr` on the route reflector.
1. Copy `sample-configs/route-reflector/daemons` and `sample-configs/route-reflector/frr.conf` to the directory `/etc/frr/`. Check the permissions and ownership afterward with `ls -l /etc/frr`. They should look like this:
```bash
-rw-r-----   1 frr  frr daemons
-rw-r-----   1 frr  frr  frr.conf
```
3. Replace placeholder values in sample configs with actual values
4. Restart `frr`

If you already configured an access point, you can verify the peering by running:

```bash
vtysh
show bgp neighbors
```

It should say that the connection with the access point is established.