# Setting Up an Access Point

This part of the guide will show you how to set up wimoved on your OpenWrt Access Point.

!!! info

    OpenWrt is available for a large number of different types of access points. Different APs may use different processor architectures. For this reason, you need a version of wimoved that is compatible with your architecture.

    We try to provide builds for as many architectures as reasonably possible. If you have OpenWrt hardware you would like to see supported, please [open an issue](https://github.com/WiMoVE-OSS/wimoved/issues/new).

!!! tip

    The AP needs to have an internet connection to be able to install dependencies

## Preparing Your AP

1. Verify that you have SSH connectivity to the access point. If not, take a look at the [OpenWrt guide for SSH](https://openwrt.org/docs/guide-quick-start/sshadministration).

!!! info

    Unless specified otherwise, all commands given in this guide should be run in the shell of the Access Point.

## Setting Up Hostapd

### Installing The Correct Version of Hostapd

Regular OpenWrt setups come with a stripped-down version of hostapd called `wpad` or `wpad-mini`. These do however not come with all features we require for our setup.

1. Uninstall the unneeded `wpad-mini`
    ```bash
    opkg update
    opkg remove wpad-mini
    opkg install hostapd
    ```
1. Reboot the Access Point

### Configuring Hostapd

OpenWrt generates the hostapd configuration file `hostapd.conf` from the OpenWrt wireless configuration.
This configuration can be edited via the web interface or by modifying the file located at `/etc/config/wireless`. We will now guide you on how to set up hostapd on your AP.

1. Go to the web interface of the AP and configure a WPA2-PSK wireless network for each radio you wish to use. Make sure to press save at the end.
1. Open the file `/etc/config/wireless` in a text editor.

    !!! tip

        OpenWrt only comes with a version of `vi` out of the box. If you are not comfortable with vim, you can also install the nano editor by running
        ```bash
        opkg update && opkg install nano
        ```

1. Go to the end of the file. There, you should find a section for each wireless network you configured in the previous step.
1. Add the following lines to each of these sections and save the file.

    ```text
    option isolate '1'
    option per_sta_vif '1'
    option vlan_file '/etc/hostapd.vlan'
    ```

1. Create a file called `/etc/hostapd.vlan` with the following content


    ```text
    *   vlan#
    ```

1. Restart the AP



When connecting to the Wi-Fi network you just created, you should see that an interface with the name `vlan*` gets created where \* is an arbitrary number. The interface should disappear, after the station disconnects. You can check the existing interfaces with `ip l`. If you do not see the interfaces, recheck that you followed the guide exactly.

## Setting Up FRR

FRRouting is used to enable the control plane of the underlying VXLAN EVPN networks. It needs to be installed on each AP and configured to talk to the route reflector in your network.

1. Install `frr` by running

    ```bash
    opkg update && opkg install frr-bgpd frr-zebra frr-watchfrr frr-vtysh
    ```

1. Create the file `/etc/frr/daemons` with the following content and replacing the placeholders with the corresponding IP addresses in your network:
```text
log syslog informational
ip nht resolve-via-default
ip6 nht resolve-via-default
router bgp 65000
  bgp router-id <YOUR AP IP>
  no bgp default ipv4-unicast
  neighbor fabric peer-group
  neighbor fabric remote-as 65000
  neighbor fabric capability extended-nexthop
  neighbor fabric ebgp-multihop 5
  ! BGP sessions with route reflectors
  neighbor <YOUR ROUTE REFLECTOR IP> peer-group fabric
  !
  address-family l2vpn evpn
   neighbor fabric activate
   advertise-all-vni
  exit-address-family
  !
!
```
1. Create the file `/etc/frr/daemons` with the following content

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

1. Restart `frr` by running `service frr restart`

You can check the configuration of FRR using the `vtysh` command. This command provides a stateful shell to manipulate FRR. More detailed documentation about vtysh can be found [here](https://docs.frrouting.org/en/latest/vtysh.html)

## Setting Up Wimoved

Wimoved is the daemon we provide that enables all Access Point features required for WiMoVE. It as well as all other dependencies need to be installed in every AP in your Wi&#8209;Fi system.

### Obtaining the Correct Binary

1. Find out which architecture your access point uses by looking it up on the [OpenWrt Table of Hardware](https://openwrt.org/toh/start).
1. Download the package for the matching architecture for your access point. There are three ways to achieve this:
      1. Download the binary from the latest release on the [Releases Page](https://github.com/WiMoVE-OSS/wimoved/releases)
      2. Download the binary from a recent pipeline run in our [GitHub Repository](https://github.com/WiMoVE-OSS/wimoved)
      3. Cross-Compile it yourself. See the [Development Guide](../../../wimoved/) for details.

### Copying Wimoved to Your AP Via SSH

!!! info

    The commands in this section need to be run on your computer, not the AP
1. Copy the package via SCP from your computer to the access point

    ```bash
    scp -O <Path to WiMoVE on your machine> root@<YOUR AP IP-ADDRESS>:
    ```

2. Login to you AP via ssh

    ```bash
    ssh root@<YOUR AP IP-ADDRESS>
    ```
    All further commands will now again need to be run on your AP.

### Installing Wimoved

1. Install the package on your access point via

    ```bash
    opkg update && opkg install <Path to wimoved>`
    ```

2. Run `wimoved`. You should see an error message, showing that wimoved was installed successfully.


### Configuring Wimoved

The wimoved config file is located at `/etc/wimoved/config`.
Before we can use the AP, we need to fill this file with some information about your installation.

1. Create a file located at `/etc/wimoved/config` and fill it with the following lines:
```text
# The location of the hostapd sockets. Leave like this for regular OpenWrt setup
hapd_sockdir=/var/run/hostapd
# Access group for hostapd sockets. Leave unchanged.
hapd_group=network
# Interval (seconds) after which interfaces for disconnected stations are removed
cleanup_interval=30
# Number of VNIs the stations are assigned to. wimoved will use VNI 1 to n.
max_vni=20
```

!!! info

    If you have any configuration needs that are not provided by the current configuration options, feel free to [open an issue](https://github.com/WiMoVE-OSS/wimoved/issues/new).

After a successful configuration, you can run `wimoved` and it should start without any errors. When connecting with a client, you should see log messages.
