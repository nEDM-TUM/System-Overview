---
title: Internal DB/Raid System
description: Overview of database/raid system on the internal nEDM network. 
layout: basic
---

## Internal DB/Raid System 

This system is a [Synology](https://www.synology.com/) RAID system (specifically RS3412RPxs), with 10 4-TB
disks.  8 of these disks provide RAID6 (total ~22 TB of space) with 2 disks
acting as hot spares.  The system runs a linux distribution
([DSM](https://www.synology.com/en-global/dsm)) and is generally administrated either from the command line (e.g. ssh) or via a [web interface](https://raid.nedm1:5001). 

![Administration screen of raid system]({{ site.baseurl }}/static/DSMAdminPage.png)

This system provides the following features:

* [Central experimental control database](#control-db), central [CouchDB](http://couchdb.apache.org/) database for experimental control and slow-control variables. 
* [OpenVPN](#openvpn), VPN access to the internal network from external locations
* [NFS/CIFS File Server](#nfs-file-server) network drive for data files 
* [File Synchronization](#file-synchronization) of files to the [cluster DB](Cluster-DB.html).
* [CouchDB Monitoring](#couchdb-monitoring), monitoring of the CouchDB database using [munin](http://munin-monitoring.org/) 

### Required Packages

Synology DSM provides software bundled in packages which allows for easy
installation.  The central server requires:

* [Cloud Station](https://www.synology.com/en-us/dsm/app_packages/CloudStation) - Automatic synchronization of files to the [Cluster DB](Cluster-DB.html). 
* [Docker](https://www.synology.com/en-us/dsm/app_packages/Docker) - 'Virtualization' program allowing deployment of 'containers' housing separate programs.
* [RADIUS Server](https://www.synology.com/en-us/dsm/app_packages/RadiusServer) - Authentication for VPN server.
* [VPN Server](https://www.synology.com/en-us/dsm/app_packages/VPNCenter) - Virtual private network

### Control DB

The control database consists of the following components:

* [CouchDB](http://couchdb.apache.org) database
* [ngingx](http://nginx.org/) reverse proxy with file server 

Both of these components run in two separate (but connected) [Docker
containers](https://www.docker.com/) on the server.  More details regarding
this system are presented on the [slow control DB page](Control-DB.html). 

### OpenVPN

An [OpenVPN server](https://openvpn.net/) runs on the central server, providing access
to the internal network from external sites.  The connection is made available
to public computers by remote forwarding an ssh tunnel (see e.g.
[here](http://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html))
to `ucgate.universe-cluster.de`.  This is done using an [`autossh` daemon on the gateway machine](GatewayMachine.html#ssh-routing).

Connection information to the OpenVPN is available on the [wiki](https://fierlinger.wiki.tum.de/Connection+to+VPN+at+Osthalle).

The only file necessary to configure this is
`/usr/syno/etc/packages/VPNCenter/openvpn/openvpn.conf` (available [on the
wiki](https://fierlinger.wiki.tum.de/Connection+to+VPN+at+Osthalle#Administration)).  Note that the two Synology Packages must be installed for this to work:

* [VPN Server](https://www.synology.com/en-us/dsm/app_packages/VPNCenter) - provides the actual OpenVPN server
* [Radius Server](https://www.synology.com/en-us/dsm/app_packages/RadiusServer) - provides authentication of users. 

### NFS File Server

This provides simply a network drive for storage of files.  Note, this should
not be confused with the [file server associated with the control database](Control-DB.html). 

### File synchronization

File sychronization happens behind the scenes using the [Cloud
Station](https://www.synology.com/en-us/dsm/app_packages/CloudStation).
This synchronizes files that are saved using the [nginx File
Server]({{ site.url }}/FileServer-Docker).  To enable this, a port is tunneled to 
`optimal.universe-cluster.de` using [autossh](GatewayMachine.html#ssh-routing).

There is also a script that runs on this server which corrects the permissions
of the files.  Otherwise these files are not transferred using CloudStation.
The script is configured in `Control Panel -> Task Scheduler` to run every hour.

{% highlight bash %}
find /volume1/Measurements/nedm -user root -type d -exec chmod 775 {} \; 
find /volume1/Measurements/nedm -user root -type f -exec chmod 664 {} \; 
find /volume1/Measurements/nedm -user root -exec chown meas_daemon:users {} \;
{% endhighlight %}

### CouchDB Monitoring 

The CouchDB server is monitored via a docker container containing a munin
process.  This tracks several metrics which can be important when monitoring
usage or tracking down problems.  The site is available
[here](http://raid.nedm1:81/). 

![Screenshot of Munin Monitoring]({{ site.baseurl }}/static/CouchDBMonitoring.png).

Setting up the monitoring requires several steps.

1. Ensure the CouchDB and associated File Server are running.  [See
here.](Central-DB.html)
2. Download (using the DSM interface) the docker container:
mgmarino/munin-docker:latest.  Run this with the execution command: `echo
"Munin Data Container"` and name the container `munin-data-container`.
(_Note_: This provides the static file structure to save the data from the
munin process.  This container should run once and exit.)
3. Start the the munin daemon container:
{% highlight bash %}
docker run -d -p 81:80\
  --volumes-from munin-data-container\
  --name munin --link nEDM-FileServer:db \
  registry.hub.docker.com/mgmarino/munin-docker:latest
{% endhighlight %}

*Note:* here we have assumed that the nEDM-FileServer ([see
here](Central-DB.html)) is running with the name: `nEDM-FileServer`.

More details about the munin container can be found 
[here]({{ site.url }}/Munin-Docker).
