# Choosing suitable parameters

Every network is different. For this reason, there are a number of parameters in the setup you will need to adapt for your use.

This page will guide you through deciding these values for your use case. You can find an example configuration below.

| Parameter name             | Example value   |
| -------------------------- | --------------- |
| Network IP range           | `10.242.0.0/16` |
| Route Reflector IP Address | `10.242.0.5`    |
| Gateway IP Address         | `10.242.0.4`    |
| Number of VNIs             | `20`            |
| VNI IPv4 Range Template    | `10.1.X.0/24`   |

## Network IP Range

All APs, gateways, and route reflectors need to talk to each other.
For this reason, you need L3 connectivity between them.
This IP range can be any size, just make sure you know what it is and that all devices inside it can reach each other at least on the following ports:

| Protocol | Port   | Purpose |
| -------- | ------ | ------- |
| TCP      | `179`  | BGP     |
| UDP      | `4789` | VXLAN   |


## Gateway IP Address

This is the IP address where the gateway can be reached.

## Route Reflector IP Address

This is the IP address at which the route reflector can be reached for both the APs and the gateway.

The functionality of the route reflector can also be included in the gateway. This is however currently only described in the Ansible deployment guide. You will need to combine the FRR configuration files for the route reflector and gateway to get this to work.

## Number of VNIs

This number decides how many virtual L2 networks will be created and devices will be split up into.
In theory, any 24-Bit number can be chosen. However, in practice, this number is limited by how many network interfaces your gateway can support.
We recommend getting started with a number around 20 and scaling up if VXLANs get too crowded.

## VNI IPv4 Range Template

Each VNI needs its own IP range that will then be used to assign IP addresses via DHCP.

For the gateway to work properly, you have to ensure that these IP ranges do not overlap each other.
Additionally, these ranges should not overlap any IP ranges the gateway has access to, i.e., the company intranet.

The exact template you decide on depends on how many VNIs you have as well as how many devices you expect to be in every L2 network.
