---
title: Gateway Machine
description: Information regarding gateway machine in the Osthalle.
layout: basic
---

## Gateway Machine

This machine provides the following functionality:

* [Firewall/router/gateway](#firewallrouter-functionality) for the internal nEDM network
* [DHCP server](#dhcp-server) for devices that require dynamic IP address
* [DNS server](#dns-server) for local devices, allowing reference via name
* Special [SSH routing](#ssh-routing) (for e.g. OpenVPN and other access)
* [Miscellaneous](#miscellaneous), system monitoring, UPS, system shutdown
* [How-To](#how-to), various instructions 

![Gateway machine in relation to the system]({{ site.baseurl }}/static/NetworkOverviewGM.png)

[CRUX](https://crux.nu/) is the linux distribution used.  This means services
are defined in: `/etc/rc.conf`.  This machine has the following IP addresses:

* `192.168.1.1` (internal network)
* `172.25.99.121` (provided by the FRM-II/Julich)

For access information, see the [wiki]({{ site.fierlingerwiki }}).  We do have
an additional device (currently unused) which is currently deployed in the Hg
tent. 

### Firewall/router functionality

This allows devices on the internal network to access the internet by routing
their requests appropriately.  The machine uses a software distribution called
[Shorewall](http://shorewall.net/) to simplify this.  The necessary files to be
edited are located in `/etc/shorewall`.  Here are a few highlighted:

* `/etc/shorewall/interfaces` - define interfaces (in our case `eth0` and `eth1`) which shorewall should use.
{% highlight bash %}
#ZONE   INTERFACE       BROADCAST       OPTIONS
loc     eth1            -               dhcp
net     eth0            -               dhcp 
{% endhighlight %}
* `/etc/shorewall/zones` - define _zones_ used by shorewall
{% highlight bash %}
#ZONE   TYPE            OPTIONS         IN                      OUT
#                                       OPTIONS                 OPTIONS
fw      firewall
net     ipv4
loc     ipv4
{% endhighlight %}
* `/etc/shorewall/policy` - define _policies_ used by shorewall.  The following only accepts from the internal network out (and to the firewall), and from the firewall to the internal network.  (To enhance security, one could also drop everything between the firewall and the internal network, but then explicit ports need to be opened, e.g. for DHCP/DNS, etc.) 
{% highlight bash %}
#SOURCE DEST    POLICY          LOG     LIMIT:          CONNLIMIT:
#                               LEVEL   BURST           MASK
loc     net     ACCEPT
loc     $FW     ACCEPT
$FW     loc     ACCEPT
$FW     net     ACCEPT
net     all     DROP            info
all     all     REJECT          info
{% endhighlight %}
* `/etc/shorewall/rules` - define the firewall rules, i.e. how the access may flow across the gateway for particular ports:
{% highlight bash %}
#ACTION         SOURCE          DEST            PROTO   DEST ... + other fields (see shorewall docs)
SECTION NEW
Ping(ACCEPT) loc $FW
Ping(DROP) net $FW
ACCEPT $FW loc icmp
ACCEPT $FW net icmp
...
{% endhighlight %}

### DHCP Server

The DHCP server (`dhcpd`) provides dynamic network addresses.  The relevant
configuration file is `/etc/dhcpd.conf`.  *Important*: this also provides
static IP addresses based upon MAC IDs, for example:
{% highlight bash %}
...
host csprobelaser.1 {
  hardware ethernet 00:19:b3:03:02:33;
  fixed-address csprobelaser.1.nedm1;
}
...
{% endhighlight %}
where the `fixed-address` is defined by the [DNS Server](#dns-server).  Another
important function that this provides is the necessary configuration for a
class of devices, including Raspberry Pis and the power supplies used in the
active coil compensation: 
{% highlight bash %}
...
class "raspberries" {
  match if ( substring(hardware,1,3) = b8:27:eb );
  default-lease-time 604800;
  option root-path "192.168.1.9:/volume1/Raspberries/boot/current,tcp,vers=3";
}

class "powersupplies" {
  match if ( substring(hardware,1,4) = 00:50:c2:E5 )
        or ( substring(hardware,1,4) = 00:50:c2:8d );
  default-lease-time 604800;
}
...
{% endhighlight %}
The configuration for the RaspPis provides a routing to the boot device for
[Raspberry Pis](Raspberry-Pis.html).

### DNS Server

The DNS server (`named`) provides name resolution for devices on the local
network: (e.g. `*.nedm1`).  The relevant configuration files are
`/var/named/revp.192.168.1` and `/var/named/zone.nedm1`.  *Both* files must be
edited and the server restarted when a device is added.  An example excerpt:

* `revp.192.168.1` 
{% highlight bash %}
...
8 			PTR armin.nedm1.
9 			PTR raid.nedm1.
9 			PTR db.nedm1.
9 			PTR data.1.nedm1.
10 			PTR sbc.1.nedm1.
11 			PTR ups.1.nedm1.
...
{% endhighlight %}
* `zone.nedm1` 
{% highlight bash %}
...
raid			A 	192.168.1.9
db			A 	192.168.1.9
data.1			A 	192.168.1.9
sbc.1			A	192.168.1.10
ups.1			A	192.168.1.11
fastxeedm.1		A	192.168.1.12
...
{% endhighlight %}

### SSH Routing

There are two `autossh` daemons that run to retain connections between the
gateway machine and other machines, including `optimal.universe-cluster.de`
(internal cluster machine) and `ucgate.universe-cluster-de` (public) machine. 

* `/etc/rc.d/autossh_init` - Provides tunneling between the DB on the internal
network and `optimal`.  This is necessary for the replication/file
synchronization that occurs between the [local DB](Internal-DB.html) and the [cluster DB](Cluster-DB.html). 

* `/etc/rc.d/autossh_init_ucgate` - Provides tunneling between the OpenVPN
server (running on the [Internal DB](Internal-DB.html) server) and the publicly
accessible `ucgate.universe-cluster.de`.

### Miscellaneous

Several additional daemons run on the gateway machine (these are defined to run
in `/etc/rc.conf`): 

* `ser_daemon` - Monitors a serial connection on the motherboard which allows
a push-button shutdown.
* `health_daemon` - Monitors general health of the network and submits to the
System Health database.  This includes information from the [UPS](UPS.html),
the [DB server](Internal-DB.html), and the gateway machine.
* `PowerChute` - Monitors the [UPS](UPS.html) and shuts down if necessary.

### How To:

* Add a new device with a static IP to the network:
  1. Define the address of the new device (currently between `.1-.100`) in the
two files: `zone.nedm1` and `revp.192.168.1` (see [above](#dns-server)).
  2. Add an entry in `dhcpd.conf` for the MAC ID of the device, referencing the
just created address.
  3. Restart the two services, `named` and `dhcpd` (must be `root`):
{% highlight bash %}
/etc/rc.d/named restart
/etc/rc.d/dhcpd restart
{% endhighlight %}

