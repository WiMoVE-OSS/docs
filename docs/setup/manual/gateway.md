# Gateway

The gateway is responsible for forwarding the internet traffic.
We use an Ubuntu server host as the basis for this setup.

!!! note

    We will provide you with some sample configurations in this chapter.
    Keep in mind that you likely have to update these for sour specific setup depending on the number of VNIs you want to use and the IP space available to you.

## Set Up FRR

1. Install `frr` on the gateway.

    ```bash
    sudo apt install frr
    ```

1. Create the file `sample-configs/gateway/frr.conf` with the following content. Remember to insert the IP of the gateway and the route reflector.

    ??? abstract "/etc/frr/daemons"

        ```text
        log syslog informational
        ip nht resolve-via-default
        ip6 nht resolve-via-default
        router bgp 65000
          bgp router-id <OWN-IP-ADDRESS>
          no bgp default ipv4-unicast
          neighbor fabric peer-group
          neighbor fabric remote-as 65000
          ! BGP sessions with route reflectors
          ! add one line per route reflector
          neighbor <IP-ADDRESS-OF-ROUTE-REFLECTOR> peer-group fabric
          !
          address-family l2vpn evpn
          neighbor fabric activate
          advertise-all-vni
          exit-address-family
          !
        !
        ```

1. Create the file `/etc/frr/daemons` with the following content

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
    sudo chmod 640 /rtc/frr/*
    ```

    !!! tip

        When you now run
        ```bash
        ls -l /etc/frr/
        ```
        you should see the following permissions:
        ```bash
        -rw-r-----   1 frr  frr daemons
        -rw-r-----   1 frr  frr  frr.conf
        ```

1. Restart `frr` by executing

    ```bash
    sudo systemctl restart frr
    ```

## Set Up VXLAN Interfaces and Bridges

We will use the Ubuntu tool `netplan` to set up and persist the required network devices. Check below for a method without the need for netplan.

1. Overwrite the file `/etc/netplan/00-installer-config.yaml` with the following content. Remember to replace the interface name for your network connection and setting the local IP address.

    ??? abstract "/etc/netplan/00-installer-config.yaml"

        ```yaml
        network:
            ethernets:
                <UPLINK IF NAME>:
                dhcp4: true
            tunnels:
                vxlan1:
                mode: vxlan
                id: 1
                local: <GATEWAY IP>
                ttl: 10
                mac-learning: false
                port: 4789

                vxlan2:
                mode: vxlan
                id: 2
                local: <GATEWAY IP>
                ttl: 10
                mac-learning: false
                port: 4789

                vxlan3:
                mode: vxlan
                id: 3
                local: <GATEWAY IP>
                ttl: 10
                mac-learning: false
                port: 4789

            bridges:
                br1:
                interfaces:
                    - vxlan1
                parameters:
                    stp: false
                addresses:
                    - 10.1.2.1/24
                br2:
                interfaces:
                    - vxlan2
                parameters:
                    stp: false
                addresses:
                    - 10.1.3.1/24
                br3:
                interfaces:
                    - vxlan3
                parameters:
                    stp: false
                addresses:
                    - 10.1.4.1/24

        ```

    !!! warning

        This setup is for a very basic gateway. Please make sure you understand what it does and change it for your needs.
        Especially, you may want to increase the number of VNIs depending on what you set in the APs.

        **Refer back to the parameters you chose for your setup.**

2. Restart the machine

    !!! tip

        In theory, using `netplan apply` should work. In our experience however, this did not give us the desired result so rebooting is the best way to go here.

??? tip "Setting up VXLANs without netplan"

    As an alternative, you can also set up the interfaces using the `ip` command directly. Note that these interfaces will not persist rebooting so you have to run the following script on every boot:

    ```bash
    #!/bin/bash
    # Copied from Vincent Bernat's blog post:
    for vni in {1..20}; do
        # Create VXLAN interface
        ip link add vxlan"${vni}" type vxlan
            id "${vni}" \
            dstport 4789 \
            nolearning
        # Create companion bridge
        brctl addbr br"${vni}"
        brctl addif br"${vni}" vxlan"${vni}"
        brctl stp br"${vni}" off
        ip link set up dev br"${vni}"
        ip a add 10.0."${vni}".1/24 dev br"${vni}"
        ip link set up dev vxlan"${vni}"
    done
    ```

## Enable IP Forwarding

1. Open the file `/etc/sysctl.conf` in an editor of your choice with root permissions:
    ```bash
    sudo nano /etc/sysctl.conf
    ```
1. Uncomment the line
   ```text
   #net.ipv4.ip_forward=1
   ```
   so that it looks like
   ```text
   net.ipv4.ip_forward=1
   ```
   to enable IPv4 forwarding
2. Uncomment the line
    ```text
    #net.ipv6.conf.all.forwarding=1
    ```
    to enable IPv6 forwarding

1. Reload the configuration file by executing
    ```bash
    sudo sysctl -p
    ```


## Set Up NFTables

1. Make sure you have `nftables` installed by running
    ```bash
    nft -v
    ```

    !!! tip

        nftables should come preinstalled with Ubuntu Server 22.04 and later

1. Create/Update the file `/etc/nftables.conf` with root privileges and paste the following content. You may want to update the `vxlans` variable to match your setup and set the uplink interface correctly.

    ??? abstract "/etc/nftables.conf"

        ```text
        #!/usr/sbin/nft -f

        # Setup for 3 VNI's
        define vxlans = { br1, br2, br3, }

        define uplink = <Uplink interface>

        flush ruleset

        # NAT for all VNIs

        table inet filter {
                chain input {
                        type filter hook input priority filter;
                }
                chain forward {
                        type filter hook forward priority filter; policy drop;
                        ct state established, related accept
                        iifname $vxlans oifname $uplink counter accept
                }
                chain output {
                        type filter hook output priority filter;
                }
                chain postrouting {
                        type nat hook postrouting priority 100; policy accept;
                        iifname $vxlans oifname $uplink masquerade
                }
        }
        ```

1. Restart the system

## Set Up Dnsmasq

1. Install `dnsmasq` as a dhcp-server on the gateway.
    ```bash
    sudo apt install dnsmasq
    ```
1. Update the file `/etc/dnsmasq.conf` with the following content. You likely need to add more entries for the correct IP ranges when you have more than 3 VNIs in use.

    !!! tip

        Refer back to the parameters you decided on for the IPv4 range for each VNI

    ??? abstract "/etc/dnsmasq.conf"

        ```text
        interface=br1
        interface=br2
        interface=br3

        bind-interfaces
        no-resolv
        dhcp-authoritative

        dhcp-range=10.0.1.100,10.0.1.200,255.255.255.0,5m
        dhcp-range=10.0.2.100,10.0.2.200,255.255.255.0,5m
        dhcp-range=10.0.3.100,10.0.3.200,255.255.255.0,5m
        ```

2. Restart dnsmasq

    ```bash
    sudo systemctl restart dnsmasq
    ```
