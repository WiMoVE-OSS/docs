# Manual Setup

This guide will show you how you can set up your own WiMoVE environment manually. Please make sure to follow the guide accurately.

!!! tip

    Since we often update our software, some details about the setup instructions might be out-of-date. In case you run into problems, feel free to take a look at the [Ansible deployment scripts](https://github.com/WiMoVE-OSS/deployment) or [Create a new Issue](https://github.com/WiMoVE-OSS/docs/issues/new).

## Getting to know the components

The parts of this guide will provide information on how to set up each component. Before getting started, please make sure you understand the purpose of each component in the whole system by reading the [Architecture page](../../architecture/index.md).


WiMoVE provides a package you can install on your OpenWRT router but needs some additional services in your network to be useful.
One of them is the route reflector and the other one is the gateway. The route reflector is part of the control plane and distributes the information between the access points and the gateway.
The gateway is responsible to terminate the overlay networks and provide internet access. We have separate Wiki entries for the setup of an [access point](https://github.com/WiMoVE-OSS/WiMoVE/wiki/Setup---Access-Point), the [route reflector](https://github.com/WiMoVE-OSS/WiMoVE/wiki/Setup---Route-Reflector) and the [gateway](https://github.com/WiMoVE-OSS/WiMoVE/wiki/Setup-gateway). If you encounter any issues during the setup, feel free to drop us a message.
