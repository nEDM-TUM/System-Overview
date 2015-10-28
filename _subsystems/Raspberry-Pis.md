---
title: Raspberry Pis 
description: How Raspberry Pis are used/programmed in the nEDM system 
layout: basic
---

## Raspberry Pis

Raspberry Pis provide much of the important functionality
available on the nEDM system.  In particular, many devices may be controlled by
scripts running on these devices.  This page documents how one can setup and
run Raspberry Pis on the local nEDM network.

### Introduction

The Raspberry Pis that run on the internal network have the following
characteristics:

* Net boot into a read-only operating system.  The net-boot ensures that all
devices run exactly the same OS and that any changes that occur to the shared
file system are propagated.  Read-only means that the devices are robust
against (un)expected power failures.
* Running scripts using [HimbeereCouch]({{ site.url }}/HimbeereCouch).  This
allows the Raspberry Pis to pull the necessary code that they should run from
the local database. 

### Setup

Setting up a new Raspberry Pi is simple.

Download the current image
[here](https://github.com/nEDM-TUM/HimbeereCouch/raw/master/nEDM/rasb_pi.img.gz)
and flash to a new SD card (can be < 64 MB).

{% highlight bash %}
wget https://github.com/nEDM-TUM/HimbeereCouch/raw/master/nEDM/rasb_pi.img.gz
gunzip -c rasb_pi.img.gz | sudo dd of=/dev/sd_card
{% endhighlight %}

where `/dev/sd_card` is the correct path to the SD card.

Then you should be able to plug in to the local nEDM network (via Ethernet) and
you're all set.

_Note_: This *must* be on the nEDM network because it relies upon access to the
[raid server's NFS](https://raid.nedm1:5001/).

### Netbooting

The set up of the NFS boot generally used instructions which were found on the
[web](http://blogs.wcode.org/2013/09/howto-netboot-a-raspberry-pi/).

The Netboot relies upon a setup in the [DHCP server on the Gateway
machine](GatewayMachine.html#dhcp-server), in particular that the following
settings are in the `dhcpd.conf` file:

{% highlight ini %}
  class "raspberries" {
    match if ( substring(hardware,1,3) = b8:27:eb );
    default-lease-time 115200;
    option root-path "192.168.1.9:/volume1/Raspberries/boot/current,tcp,vers=3";
  }
{% endhighlight %}

which ensures that the Raspberry Pis use the given path as their root file path.

### Running

As noted, all Raspberry Pis mount the NFS drive read-only.  To change (e.g.
update or install new software), then the drive must re-mounted rw.  After one
logs in as `root` (see the [wiki]({{ site.fierlingerwiki }})), then the `ro`
(read-only) and `rw` (read-write) alias commands should be available.  A normal
update would look like: 

{% highlight bash %}
rw # Make read-write
# ... Perform updates to the file system
# ...    install new software, etc.
ro # Make read-only
{% endhighlight %}

The aliases are (for reference):
{% highlight bash %}
alias rw='mount -o remount,rw /'
alias ro='mount -o remount,ro /'
{% endhighlight %}

Note that this will automatically propagate to the other systems.  It could be,
however, that the devices need to be restarted either using the command
`reboot` or by pulling and reinserting the power cable (safe on the read-only
system).

### `himbeerecouch`/`pynedm`

`himbeerecouch` is a python module, installed on the Netboot
distribution, which is used by the Raspberry Pis to pull the code from the 
[database](Control-DB.html) that should be run.  See [the documentation]({{ site.url }}/HimbeereCouch)
for more details about how this works.  To update the software, use the
command:

{% highlight bash %}
/etc/init.d/rspby install
{% endhighlight %}

`pynedm` is a [software package]({{ site.url }}/Python-Slow-Control) which
implements much of the functionality needed in the nEDM experiment.  It is
installed on the Raspberry Pis since it is typically used in scripts pulled
down by `himbeerecouch`. To update `pynedm` on the Raspberry Pis:

{% highlight bash %}
/etc/init.d/rspby install_py
{% endhighlight %}

The `himbeerecouch` daemon is run using `supervisor` and configured via the
file (`/etc/supervisor/conf.d/raspberry.conf`):

{% highlight ini %}
[program:raspberry]                                                                                        
command=python -c 'import himbeerecouch.prog as p; p.run("/etc/rspby/server")'                             
autostart=true                                                                                             
autorestart=true                                                                                           
stopsignal=INT                                                                                             
redirect_stderr=true                                                                                       
stdout_logfile=/var/log/rspby_daemon.log 
{% endhighlight %}

### Terminal and log servers

There is a [terminado server](https://github.com/takluyver/terminado) running
on the Raspberry Pis which allows visualization and interaction with the
terminal via the [web
interface](http://db.nedm1/page/control/nedm/raspberries).

These are two daemons which are configured via:

* `/etc/supervisor/conf.d/log.conf`:
{% highlight ini %}
[program:rspy_log]                                                                                         
command=sudo -u daq sh -c 'cd /home/daq/terminado_server && python log.py'                                 
autostart=true                                                                                             
autorestart=true                                                                                           
stderr_logfile=/var/log/rspby_log.err                                                                      
stdout_logfile=/var/log/rspby_log.log   
{% endhighlight %}
* `/etc/supervisor/conf.d/terminado.conf`:
{% highlight ini %}
[program:terminado]                                                                                        
command=sudo -u daq sh -c 'cd /home/daq/terminado_server && python server.py'                              
autostart=true                                                                                             
autorestart=true                                                                                           
stderr_logfile=/var/log/terminado.err                                                                      
stdout_logfile=/var/log/terminado.log                                                                      
{% endhighlight %}

