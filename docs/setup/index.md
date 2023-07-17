# Setup Guide

## Requirements

Before getting started, please make sure you have all hardware required for a working WiMoVE setup.
You need:

- At least one Access Point running OpenWrt 22.03 or later
- One Ubuntu Server 22.10 or later machine to be used as a Route Reflector
- One Ubuntu Server 22.10 or later machine to be used as a Gateway

!!! info

    The Route Reflector and Gateway can be combined into one machine. See the Ansible deployment guide for details.

## Installation Methods

There are two different ways for setting up WiMoVE on your own hardware.

### Setup using Ansible (Recommended)

Since there are a number of services that need to be set up, we recommend using the Ansible installation method.

[Install using Ansible](ansible_setup.md){ .md-button }


### Manual Setup

Alternatively, you can also set up all the components manually.

!!! warning

    There are a number of tasks in this process that can be extremely repetitive (i.e., setting up interfaces, provisioning multiple routers).
    For this reason, we only recommend this option if you just want test WiMoVE in a small setup and are willing to try around a bit.

[Install manually](manual/){ .md-button }
