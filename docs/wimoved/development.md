# Development Setup

A possible workflow is to update the code of WiMoVE and use the script `scripts/build_openwrt.sh` to build it. Install the resultig package on an access point. This can be pretty time consuming for many small changes. After following this guide, you can compile WiMoVE directly for your host system. Currently compiling hostapd for your host system is only possible on Linux.

### Setup hostapd

1. The build of hostapd has to support vlans. Check this if you encounter issues with your installation.
1. Use the sample config in `sample-configs/development/hostapd.conf`.
1. Replace the placeholder values in the config.
1. Create a file `/etc/hostapd/hostapd.vlan` with the content: 
```
*   vlan#
```
5. Restart hostapd.

### Build WiMoVE yourself

1. Clone the repo.
1. Install `libnl`, `libnl-route` [Repo](https://github.com/thom311/libnl) and `prometheus-cpp` [Repo](https://github.com/jupp0r/prometheus-cpp). The best installation method for those dependencies depends on your platform. For many platforms packages for `libnl` and `libnl-route` might be provided. `prometheus-cpp` has to be built according to the README.
1. Build the project by running `cmake .` followed by `make -j$(nproc)`.
1. You now have the WiMoVE binary for your architecture.

This binary probably won't work on your access point because it has a different architecture. You have to use cross compiling to get a compatible binary.

### Formatting

Format the source files by running

```bash
find src -iname *.h -o -iname *.cpp | xargs clang-format -i