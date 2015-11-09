---
title: Single-board computers (SBCs)
description: Overview of single-board computers used in the nEDM experiment
layout: basic
---

SBCs are used in the nEDM experiment to:

* readout over the VME bus
* communicate with [ORCA](http://orca.physics.unc.edu/)
* Communicate with the CASCADE detectors.

All SBCs run a read-only linux system (similar to the
[Raspberry Pis](Raspberry-Pis.html)) that allows them to be safely removed from
power at all times.


### Backup images

* ORCA - Active Coil Compensation/Struck readout: a backup image for this SBC is here ([Cluster server](http://10.155.59.88/_attachments/nedm%2Fsystem_health/sbc_backup_cards/sbc.2.backup.gz) ) ([Osthalle server](http://db.nedm1/_attachments/nedm%2Fsystem_health/sbc_backup_cards/sbc.2.backup.gz)).

* CASCADE detectors - A backup image for this SBC is here ([Cluster server](http://10.155.59.88/_attachments/nedm%2Fsystem_health/sbc_backup_cards/sbc.cascade.backup.gz) ) ([Osthalle server](http://db.nedm1/_attachments/nedm%2Fsystem_health/sbc_backup_cards/sbc.cascade.backup.gz)).

Both of these can be flashed to a CompactFlash card:
{% highlight bash %}
gunzip -c sbc.backup.gz | sudo dd of=/dev/sd_card
{% endhighlight %}

Note that after this, the file system on the second partition will have to be
rebuilt, which can be done on a linux machine that has access to the
CompactFlash card.  For example, if the card is on `/dev/sbc`, this can be done:

{% highlight bash %}
mkfs.ext3 /dev/sbc2
{% endhighlight %}

### Updating the software on a running system

After logging in to the SBC as as `root` (see the [wiki]({{ site.fierlingerwiki
}})), the `ro` (read-only) and `rw` (read-write) alias commands should be
available.  A normal system update would look like:

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



