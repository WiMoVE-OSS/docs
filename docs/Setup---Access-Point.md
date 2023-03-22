## Installing WiMoVE

1. Find out what architecture your access point is by looking it up on the OpenWRT wiki.
1. Download the package for the matching architecture for your access point. You find the package in the artifacts of the build pipeline or you can cross-compile it yourself.
1. Verify that you have ssh connectivity to the access point. [OpenWRT guide for ssh](https://openwrt.org/docs/guide-quick-start/sshadministration)
1. Copy the package via SCP from your computer to the access point `scp -O <Path to WiMoVE on your machine> <access-point>`
1. Login via ssh on to your access point.
1. Install the package on your access point via `opkg update && opkg install <Path to WiMoVE>`. An internet connection is required for this step in order to fetch dependencies.
1. Run `wimove`. You should see an error message, which shows that wimove is successfully installed.

## FRR

1. Install `frr` via `opkg update && opkg install frr-bgpd frr-zebra frr-watchfrr frr-vtysh`
1. Copy `sample-configs/access-points/daemons` and `sample-configs/access-points/frr.conf` to the directory `/etc/frr/`.
1. Replace placeholder values in sample configs with actual values. Placeholder values are marked with `<>`.
1. Restart `frr` by running `service frr restart`

You can check the configuration of FRR with the `vtysh` command. This command provides a stateful shell to manipulate FRR. More detailed documentation about vtysh can be found [here](https://docs.frrouting.org/en/latest/vtysh.html)

## Hostapd

The hostapd config will be generated for you from `/etc/config/wireless`.

1. It is important that the full version of hostapd is installed. The default package for hostapd does *not* work with our software. The correct package is named `hostapd`
1. Configure a Wi-Fi network for the access point with `WPA-PSK` security via the web interface.
1. The file `/etc/config/wireless` should now contain a `wifi-iface` section. The following options are required in this section. 
```text
    option isolate '1'
    option per_sta_vif '1'
    option vlan_file '/etc/hostapd.vlan'
```
4. Create the file `/etc/hostapd.vlan` with the following content:
```text
*   vlan#
```
5. Restart the access point

When connecting to the Wi-Fi network you just created, you should see that an interface with the name `vlan*` gets created where * is an arbitrary number. The interface should disappear, after the station disconnects. You can check the existing interfaces with `ip l`. If you do not see the interfaces, recheck that you followed the guide exactly.

## Configure WiMoVE

The default path for WiMoVE config is `/etc/wimove/config`. It can be configured as the first command line argument of WiMoVE. A default configuration file is available at `sample-configs/access-points/wimove`. There are listed all configuration options with the default values and an explanation. Right now there is almost no configuration. If you miss any configuration option, please feel free to open an issue.

After a successful configuration, you can run `wimove` and it should start without any errors. When connecting with a client, you should see log messages.