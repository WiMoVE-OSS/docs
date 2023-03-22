# Home

In large Wi-Fi systems, broadcast traffic can take up large amounts of airtime[^1].
Partitioning a Wi-Fi system into multiple subnets breaks handover while solutions like broadcast suppression break user functionality.

**WiMoVE** is a scalable Wi-Fi System that partitions stations into overlay L2 domains to limit the amount of wireless L2 broadcast traffic.
Overlay L2 domains "follow" the stations, being resized on demand, thus preserving handover.

WiMoVE is built with standard network protocols, on top of open-source technology:

- The overlay networks use **BGP EVPN** with **VXLAN** encapsulation.
- All BGP speakers run **FRR** to exchange EVPN routes.
- The access points run **OpenWRT** with a custom, open-source daemon called `wimoved`.

This solution allows for using commodity access points running OpenWRT for large-scale Wi-Fi deployments, even from different vendors.

## About

WiMoVE is a bachelor's project at [Hasso-Plattner-Institute](https://hpi.de) at the chair for [Internet Technologies and Softwarization](https://hpi.de/en/research/research-groups/internet-technologies-and-softwarization.html).

The project was carried out by [Aaron Schlitt](https://aaronschlitt.de/), [Alexander Sohn](https://asohn.de/), [Lina Wilske](https://github.com/linascience) and [Richard Wohlbold](https://rgwohlbold.de) under the supervision of [Holger Karl](https://hpi.de/karl/people/holger-karl.html) in partnership with [BISDN GmbH](https://www.bisdn.de/).


[^1]: See <https://www.youtube.com/watch?v=v8y-r9JBhmw> for a talk on this issue.