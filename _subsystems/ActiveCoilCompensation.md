---
title: Active Coil Compensation
description: Software to run the Active Coil Compensation
layout: basic
order: 100
---

The Coil Compensation uses an open-source, Mac OS X-based software called
[ORCA](http://orca.physics.unc.edu/).  We use a specific fork of the software
to control the coil compensation system.

### Fresh install

Doing a fresh install requires access to the private Orca
[repository fork](https://github.com/nEDM-TUM/Orca).  First obtain access to
this (became a member of the nEDM-TUM organization).  Then one can do the
following:

{% highlight bash %}
git clone git@github.com:nEDM-TUM/Orca.git
cd Orca
open Orca.xcodeproj # This should open XCode
{% endhiglight %}

Once in Xcode, you should simply be able to build.

The current experiment file used is available here
([Cluster server](http://10.155.59.88/_attachments/nedm%2Factive_coil_compensation/experiment_backup/ActiveCoilCompensation.Orca) ) ([Osthalle server](http://db.nedm1/_attachments/nedm%2Factive_coil_compensation/experiment_backup/ActiveCoilCompensation.Orca)).
