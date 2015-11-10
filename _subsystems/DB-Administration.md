---
title: Administration
description: List of administration tasks for systems in the nEDM experiment.
layout: basic
---

List of administration tasks that are commonly performed for the nEDM systems:

* Administration of main linux servers (this includes software updates, etc.):
  - [Osthalle DB/Server](Internal-DB.html), [Admin interface](https://raid.nedm1:5001)
  - [Cluster DB/Server](Cluster-DB.html), [Admin interface](https://10.155.59.88:5185)
  - [Gateway Machine](GatewayMachine.html)
    * Update/add devices to network
    * Update DHCP settings
  - [UPS](UPS.html), [Admin interface](http://ups.1.nedm1/)
* Administration of network, including network switches
  - 2 unmanaged, 1 managed: [switch.1](http://switch.1.nedm1/)
* Administration of [GitHub group repository](https://github.com/nedm-tum)
  - Pushing of production software to [DB interface]({{ site.url }}/nEDM-Interface).
* Monitoring
  - [Local DB server](Internal-DB.html#couchdb-monitoring), [Admin interface](http://raid.nedm1:81/)
  - Usage on both Osthalle and Cluster DBs.
* Raspberry Pis 
  - [Setting up new devices](Raspberry-Pi.html#setup)
