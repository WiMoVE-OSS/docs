# Home

In large Wi&#8209;Fi systems, broadcast traffic can take up large amounts of airtime[^1].
Partitioning a Wi&#8209;Fi system into multiple subnets breaks handover while solutions like broadcast suppression break user functionality.

**WiMoVE** is a scalable Wi&#8209;Fi System that partitions stations into overlay L2 domains to limit the amount of wireless L2 broadcast traffic.
Overlay L2 domains "follow" the stations, being resized on demand, thus preserving handover.

WiMoVE is built with standard network protocols, on top of open&#8209;source technology:

- The overlay networks use [BGP EVPN](https://www.rfc-editor.org/rfc/rfc8365.html) with [VXLAN](https://www.rfc-editor.org/rfc/rfc7348) encapsulation.
- All BGP speakers run [FRRouting](https://frrouting.org/) to exchange EVPN routes.
- The access points run [OpenWrt](https://openwrt.org/) with a custom, open&#8209;source daemon called [WiMoVEd](https://github.com/wimove-oss/wimoved).

This solution allows for using commodity access points running OpenWrt for large&#8209;scale Wi&#8209;Fi deployments, even from different vendors.

A high-level explanation of the project can be seen in [this presentation](https://www.tele-task.de/lecture/video/10107/) (german).


## Where to go from here

[Architecture Overview](architecture/){ .md-button }
[Design Document](design/){ .md-button }
[Setup](setup/){ .md-button }

## About

WiMoVE is a bachelor's project at [Hasso&#8209;Plattner&#8209;Institute](https://hpi.de) at the chair for [Internet Technologies and Softwarization](https://hpi.de/en/research/research-groups/internet-technologies-and-softwarization.html).

The project was carried out by [Aaron Schlitt](https://aaronschlitt.de/), [Alexander Sohn](https://asohn.de/), [Lina Wilske](https://github.com/linascience) and [Richard Wohlbold](https://rgwohlbold.de) under the supervision of [Holger Karl](https://hpi.de/karl/people/holger-karl.html) in partnership with [BISDN GmbH](https://www.bisdn.de/).


[^1]: See <https://www.youtube.com/watch?v=v8y-r9JBhmw> for a talk on this issue.
