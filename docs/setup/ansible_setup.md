# Deployment Using Ansible

This guide will show you how you can install you own WiMoVE setup using our ansible playbook. The playbook is designed to set up all components (APs, gateway, route reflector). It can however also be used to only set up parts of these components.

!!! tip "Before you get started"

    It is a very good idea to at least skim over the different steps of this guide so you know what to expect *before* you try to set up your own system.

## Getting to know the components

The parts of this guide will provide information on how to set up each component. Before getting started, please make sure you understand the purpose of each component in the whole system by reading the [Architecture page](../../architecture/index.md).


WiMoVE provides a package you can install on your OpenWRT router but needs some additional services in your network to be useful.

One of them is the route reflector and the other one is the gateway. The route reflector is part of the control plane and distributes the information between the access points and the gateway.

The gateway is responsible for terminating the overlay networks and provide internet access.

## Clone the Repository

Before we get started, please clone the deployment repository from our GitHub:

```bash
git clone https://github.com/WiMoVE-OSS/deployment.git
```

## Configure the Inventory

First, we have to set up our inventory.
This file decides which devices will be provisioned by the ansible playbook.

To get started. rename the file `example-inventory.yml` to `inventory.yml`.

!!! warning "Warning: A note on IP ranges"

    This playbook is designed to use the IP range `10.0.0.0/8` for the virtual L2 networks. If you use (part of) this IP range for your network already, consider modifying the dnsmasq and VXLAN configuration in the gateway folder to suit your requirements.

### Set WiFi Parameters

First things first, we need to set a name for our wireless network. Put this name in the `ssid` variable. The passphrase for the network goes into the `secret` variable.

If you want to use 802.11r fast transition, you have to set the `fast_transition` variable to 1.

!!! warning

    In our testing, we experienced some issues with using FT on some of our APs (especially ZyXel NWA50AX). So make sure your hardware (both AP and stations) support this feature before you enable it in a production environment.

### Set a Log Server

Our ansible playbook sets up `syslog-ng` as a syslog backend. It supports forwarding all logs to a log server.
This has proven to be quite useful in our testing setups.
If you want to set up a logging server for yourself, we recommend getting started with [this repository](https://github.com/lux4rd0/grafana-loki-syslog-aio){:target="_blank"}.

If you do not want to use this feature, simply set `127.0.0.1` as the parameter for `log_server`, otherwise put the IP/domain of your log server here.

### Set Route-Reflector Configuration

Please set the IP address of the route reflector in the variable `route_reflector_ip`. Additionally, please set an IP range from which the route reflector should accept connections (i.e. `10.242.0.0/16`) in the `listen_range` variable.

### Set Gateway Configuration

For the gateway, we need to additionally configure the names of the network interfaces that are used.

- `uplink_if` is the interface that will be used for internet access for the overlay networks
- `wimove_if` is the interface that will be connected to the L3 environment where all APs can be reached

By default, we assume that the same interface is used for both internet access and the APs. If this is not the case, uncomment and edit the `wimove_if` line in the inventory.

### Set Number of VNIs

We lastly have to decide how many virtual L2 networks we want to provide.

The ansible playbook theoretically supports up to 65.000 VNIs but we recommend starting with a small number (i.e. 20) to get started and scale up later.
The reason for this is the fact that the large number of network interfaces on the gateway can take up lots of resources which you likely do not want in a testing scenario.

### Setting Hosts

Take a look at the hosts section of the inventory file.
There, you can see a number of groups. For route reflector and gateway, please insert the corresponding IP addresses.

For the AP section, please put the different types of APs with their names and IP addresses in the inventory.

## Deploy to different APs

The ansible playbook currently supports two different types of APs. `zyxel` and `linksys`. If you want to have more or other models, you just have to create the directories and in `roles/access-point/{files,templates}/model-name`. Then you can add a new group to the children of the group `openwrt` in the inventory and set the `directory` variable accordingly. Then follow the next steps.
## Gather Binaries

1. Find out which architecture your access point uses by looking it up on the [OpenWRT Table of Hardware](https://openwrt.org/toh/start).
1. Download the package for the matching architecture for your access point. There are three ways to achieve this:
      1. Download the binary from the latest release on the [Releases Page](https://github.com/WiMoVE-OSS/wimoved/releases)
      2. Download the binary from a recent pipeline run in our [GitHub Repository](https://github.com/WiMoVE-OSS/wimoved)
      3. Cross-Compile it yourself. See the [Development Guide](../../../wimoved/) for details.
1. Place the binary in the corresponding folder for your AP type. The filepath is`roles/access-point/files/model-name`. Make sure that it has the file extenstion `.ipk` and it is the only `.ipk` file in that directory.

!!! tip

    You can use arbitrary model names. The `directory` variable for a host only has to match the directory name in `roles/access-points/files` to get the correct binary.

## Generate openWRT configuration files

Since each AP is different, we need different kinds of configuration files.
For this reason, we do not provide configuration files for your APs but rather guide you how to create them yourself.

1. Go to the web interface of one of your APs and configure a WPA2-PSK wireless network for each radio you wish to use. You can set any SSID and password, we will overwrite them in a minute. Make sure to press save at the end.
1. Open an SSH console to the AP and run

    ```bash
    uci export
    ```
    to export the configuration for the AP.

1. Copy the output of the command and place it in a file called `config.ota.j2` in the folder `roles/access-point/templates/model-name` that corresponds to your AP type in the ansible playbook.
1. Open the file in a text editor.
1. Go to the end of the file. There, you should find a section for each wireless network you configured in the previous step.
1. Adjust each of these `wifi-iface` sections so that they contain the following lines:

    ```text
    option ssid '{{ ssid }}'
    option isolate '1'
    option key '{{ secret }}'
    option ieee80211r '{{ fast_transition }}'
    option mobility_domain '0101'
    option reassociation_deadline '1000'
    option ft_over_ds '{{ ft_over_ds }}'
    option ft_psk_generate_local '1'
    option per_sta_vif '1'
    option vlan_file '/etc/hostapd.vlan'
    ```
    !!! warning

        Make sure that there are no two lines with the same name in each of the `wifi-iface` sections of the file. Otherwise, openWRT will later not accept the configuration.

## Installing Dependencies for Ansible and Running the Playbook

### Install Dependencies

We use an ansible plugin to provision openWRT. This plugin needs to be installed by running
```bash
ansible-galaxy install -r requirements.yml
```
in the repo folder.

### Preparing the Run

Before we run the playbook, we have to make sure that we can connect to all APs and other machines via SSH.
For this to work properly, you have to set up SSH key authentication on each of the devices.

!!! tip

    To copy your SSH key to your APs, you can either use the openWRT web interface or the typical `ssh-copy-id` command.

### SSH setup

Please make sure that your SSH config file can resolve all the names you gave to the hosts in your inventory. Otherwise, when running the playbook you will get an error that the name could not be resolved.

### Running the playbook

!!! warning "Before you run the playbook: A note on SSH key passphrases"

    Ansible attempts to open multiple SSH connections over the process of running a playbook. If you have configured a passphrase for your private key (as is generally a good idea), this will result in many password prompts.

    To prevent this, we recommend using `ssh-agent` to cache the unencrypted private key in the current terminal.

    To get started, open the terminal in which you want to run ansible later.
    Then execute
    ```bash
    eval "$(ssh-agent -s)"
    ```
    to start ssh-agent. Now, we need to load your private key by running (depending on the location of your key file)
    ```bash
    ssh-add ~/.ssh/id_rsa
    ```
    This will prompt you for the passphrase of your key.

To finally run the playbook, execute
```bash
ansible-playbook -i inventory.yml install.yml -K
```
