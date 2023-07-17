# Setting Up a route reflector

In addition to the access points, we use a route reflector to build our control plane. A route reflector can be any kind of machine where FRR can be installed. It just needs a L3 connection to the access points and the gateway. We again use an Ubuntu server host for this tutorial.

## Setting Up FRR

!!! warning

    This part of the guide is very similar to the one for the gateway. However, the configuration files for FRR are different in this case.

1. Install `frr` on the gateway.

    ```bash
    sudo apt install frr
    ```

2. Create the file `sample-configs/gateway/frr.conf` with the following content. Remember to insert the IP of the route reflector.

    ??? abstract "/etc/frr/daemons"

        ```text
        log syslog informational
        ip nht resolve-via-default
        ip6 nht resolve-via-default
        router bgp 65000
          bgp router-id <OWN-IP-ADDRESS>
          bgp cluster-id <OWN-IP-ADDRESS>
          bgp log-neighbor-changes
          no bgp default ipv4-unicast
          neighbor fabric peer-group
          neighbor fabric remote-as 65000
          neighbor fabric capability extended-nexthop
          neighbor fabric ebgp-multihop 5
          neighbor fabric update-source <OWN-IP-ADDRESS>
          bgp listen range <ALL-AP-SUBNETS-CIDR-NOTATION> peer-group fabric
          !
          address-family l2vpn evpn
          neighbor fabric activate
          neighbor fabric route-reflector-client
          exit-address-family
          !
        !
        ```

3. Create the file `/etc/frr/daemons` with the following content

    ??? abstract "/etc/frr/daemons"

        ```text
        bgpd=yes
        ospfd=no
        ospf6d=no
        ripd=no
        ripngd=no
        isisd=no
        pimd=no
        ldpd=no
        nhrpd=no
        eigrpd=no
        babeld=no
        sharpd=no
        pathd=no
        pbrd=no
        bfdd=no
        fabricd=no
        vrrpd=no


        vtysh_enable=yes
        zebra_options="  -A 127.0.0.1 -s 90000000"
        bgpd_options="   -A 127.0.0.1"
        ospfd_options="  -A 127.0.0.1"
        ospf6d_options=" -A ::1"
        ripd_options="   -A 127.0.0.1"
        ripngd_options=" -A ::1"
        isisd_options="  -A 127.0.0.1"
        pimd_options="   -A 127.0.0.1"
        ldpd_options="   -A 127.0.0.1"
        nhrpd_options="  -A 127.0.0.1"
        eigrpd_options=" -A 127.0.0.1"
        babeld_options=" -A 127.0.0.1"
        sharpd_options=" -A 127.0.0.1"
        pbrd_options="   -A 127.0.0.1"
        staticd_options="-A 127.0.0.1"
        bfdd_options="   -A 127.0.0.1"
        fabricd_options="-A 127.0.0.1"
        vrrpd_options="  -A 127.0.0.1"
        ```

1. Ensure these configuration files are owned by the `frr` user and group by running

    ```bash
    sudo chown -R frr:frr /etc/frr/
    ```

1. Ensure that only the `frr` user has read and write permissions to the file and the `frr` group can read the file. All other users should not have any access. To achieve this, run

    ```bash
    sudo chmod 640 /etc/frr/*
    ```

1. Restart `frr` by executing

    ```bash
    sudo systemctl restart frr
    ```

## Verifying the Setup

If you already configured an access point, you can verify the peering by running:

```bash
$ sudo vtysh
rr# show bgp neighbor
```

It should say that the connection to the access point has been established.
