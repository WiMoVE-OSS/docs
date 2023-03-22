# Architecture

In this section we explain the different components of our system on a high level with an exmaple. In this scenario we have four access points and four connected stations. Those four stations are partitioned in three overlay networks. Every station is connected to a different access point.

![Overlay Networks before Roaming](../images/architecture/before_roam.png)

The different colors show the different overlay L2 domains the stations are assigned. An overlay L2 domain consists of a small number of stations and one gateway that provides L3 connectivity to other networks. To which overlay network a station is assigned is determined by its MAC address. As an advanced feature set we might add an optional central service, that is responsible for the assignment. Because an access point is only part of those overlay networks it has a station connected in, after a handover the overlay L2 domains might change and look like this:

![Overlay Networks after Roaming](../images/architecture/after_roam.png)

The system is fully transparent to the stations. It has normal unicast, multicast and broadcast L2 capabilities in the overlay networks. 

We use BGP EVPN as a control plane with a central route reflector.

![BGP peerings](../images/architecture/bgp_peerings.png)

The black lines are iBGP peerings. The route reflector distributes received BGP updates to all other peers. It is possible to use BGP topologies without a central route reflector or with multiple route reflectors, but we haven't tested such topologies yet.

A more in depth explanation of our design decisions is available in our [design document](Link to desgin document). 
