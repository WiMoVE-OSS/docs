# wimoved

`wimoved` is the Access Point daemon for WiMoVE.

## Requirements

`wimoved` has the following dependencies: 

- rtnetlink for interface manipulation
- `hostapd` for event ingestion

## Test Systems

We have tested `wimoved` on the following systems:

- Development setup on [Arch Linux](https://archlinux.org/) (`amd64`)
- OpenWRT 22.03.2 on the following architectures:
    - `ramips-mt7621` on `ZyXEL NWA50AX`
    - `mvebu-cortexa9` on `Linksys WRT1900ACS`