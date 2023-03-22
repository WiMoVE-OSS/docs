The gateway is responsible for forwarding the internet traffic. We use an Ubuntu server host. We provide an ansible playbook, that generates a netplan config, a dnsmasq config and the nftables rules for a given number of VXLANs. This playbook will only work with an Ubuntu host and might need some tweaking to fit your use case.

#### FRR

1. Install `frr` on the gateway.
1. Copy `sample-configs/gateway/daemons` and `sample-configs/gateway/frr.conf` to the directory `/etc/frr/`. Check the permissions and ownership afterward with `ls -l /etc/frr`. They should look like this:

```bash
-rw-r-----   1 frr  frr daemons
-rw-r-----   1 frr  frr  frr.conf
```

1. Replace placeholder values in sample configs with actual values
1. Restart `frr`

#### VXLAN interfaces

1. Create all vxlan interfaces with the script `sample-configs/gateway/setup_vxlan.sh`. The script creates one bridge and one vxlan interface for every vni.

An alternative approach to using this script is using `netplan`. An advantage is that the interfaces will be created at boot. A sample netplan configuration can be found in `sample-configs/gateway/netplan.config`.

#### Enable IP forwarding

1. Execute `sudo sysctl -w net.ipv4.ip_forward=1` to allow IPv4 forwarding.
1. Execute `sudo sysctl -w net.ipv6.conf.all.forwarding=1` to allow IPv6 forwarding.

#### NFtables

1. Install `nftables`
1. Copy `sample-configs/gateway/nftables.conf`. Adapt the variables vxlans and uplinks according to your setup.

#### Dnsmasq

1. Install `dnsmasq` as dhcp-server on the gateway.
1. Copy dnsmasq config. `sample-configs/gateway/dnsmasq.conf`. This is a config file matching the default setup of `sample-configs/gateway/setup_vxlan.sh`. If you change anything within this script, the config probably has to be updated.
1. Restart dnsmasq.