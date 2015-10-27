---
title: UPS 
description: Information regarding the UPS
layout: basic
---

## Uninterruptible Power Supply

The UPS is an [Smart-UPS RT 6000 RM XL](http://www.apc.com) which is capable of
supporting up to roughly 25 Amps of equipment.  Currently a set of critical
devices (this list may not be actual) are connected: 
  
* [Raid/DB Server](Internal-DB.html)
* Mac Mini - Active compensation coil control 
* Mac Mini - Cs control 
* [Gateway machine](GatewayMachine.html)
* Various Cs devices

The UPS is available at the address [http://ups.1.nedm1](http://ups.1.nedm1)
(available when connected with the [OpenVPN](Internal-DB.html#openvpn)).  See
the [wiki]({{ site.fierlingerwiki }}) for login details.

![UPS Home Page]({{ site.baseurl }}/static/UPSPage.png)

The state of the UPS device is monitored in the [System Health
DB](http://db.nedm1/page/monitor/nedm/system_health), including the current
draw on the device.  These variables should *always* be monitored when adding a
new device to the UPS. 

