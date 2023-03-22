# Development Setup

This guide shows how to compile `wimoved` for your host system.
We have only tested the development setup on Linux.

## Setup hostapd

First, we will set up [hostapd](https://w1.fi/).

- Install `hostapd` using your distribution's package manager. Alternatively, you can build it from source, see [here](https://w1.fi/). By default, hostapd comes with VLAN support. If you encounter issues, make sure that it was compiled with `CONFIG_NO_VLAN=n`.
- Put the following in your `/etc/hostapd.conf`, replacing the placeholder values:

??? abstract "/etc/hostapd.conf"

    ```
    interface=<interface>
    ssid=<ssid>
    ieee80211d=1
    country_code=<Your country code>
    hw_mode=g
    ieee80211n=1
    channel=6
    beacon_int=1000
    dtim_period=2
    max_num_sta=255
    rts_threshold=-1
    fragm_threshold=-1
    macaddr_acl=0
    auth_algs=1
    ignore_broadcast_ssid=0
    wmm_enabled=0
    eapol_key_index_workaround=0
    eap_server=0
    wpa=2
    wpa_key_mgmt=WPA-PSK
    rsn_pairwise=CCMP
    wpa_passphrase=<Secret key>
    per_sta_vif=1
    vlan_file=/etc/hostapd/hostapd.vlan
    ```

- Create a file `/etc/hostapd/hostapd.vlan` with the following content: 

???+ abstract "/etc/hostapd.vlan"

    ```
    *   vlan#
    ```

- Start `hostapd`, either with `systemctl start hostapd` or on the command line.

!!! tip

    You might need to stop `NetworkManager` before starting `hostapd` since the programs interfere with each other.

## Build WiMoVE

- Install [`libnl`](https://github.com/thom311/libnl). On a recent Linux system, the corresponding package is probably already installed.
- Install [`prometheus-cpp`](https://github.com/jupp0r/prometheus-cpp).
    - On Ubuntu, install the package `prometheus-cpp-dev`.
    - On Arch Linux, the package is available in the AUR as `prometheus-cpp-git`.
- Clone the repository by running `git clone https://github.com/WiMoVE-OSS/wimoved`.
- Build the project by running `cmake .` followed by `make -j$(nproc)`.
- Create a configuration file `wimoved.conf`

???+ abstract "wimoved.conf"

    ```
    hapd_sock=/var/run/hostapd/<interface>
    ```

- Start `wimoved` by running `./wimoved wimoved.conf`.

To run `wimoved` on an Access Point, you have to cross-compile.
See [Cross Compiling](./cross_compiling.md) for more information.

## Formatting

Format the source files by running

```sh
find src -iname *.h -o -iname *.cpp | xargs clang-format -i
```