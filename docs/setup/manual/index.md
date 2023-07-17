# Manual Setup

This guide will show you how you can set up your own WiMoVE environment manually. Please make sure to follow the guide accurately.

!!! tip

    Setting up WiMoVE manually can be quite a complex task. In case you run into problems, feel free to take a look at the [Ansible deployment scripts](https://github.com/WiMoVE-OSS/deployment) or [Create a new Issue](https://github.com/WiMoVE-OSS/docs/issues/new).

!!! tip "Before you get started"

    It is a very good idea to at least skim over the different steps of this guide so you know what to expect *before* you try to set up your own system.

## Getting to Know the Components

The parts of this guide will provide information on how to set up each component. Before getting started, please make sure you understand the purpose of each component in the whole system by reading the [Architecture page](../../architecture/index.md).


WiMoVE provides a package you can install on your OpenWrt router but needs some additional services in your network to be useful:

- The **gateway** is responsible for terminating the overlay networks and provide internet access.
- The **route reflector** is part of the control plane. It receives routing information and distributes the information to the access points and the gateway.

## Deciding on Parameters

There are a number of parameters you will need to decide on when you want to set up your own WiMoVE system.

Please take a look at the following page and decide on parameters before you get going.

[Parameter Guide](./parameters){ .md-button }
