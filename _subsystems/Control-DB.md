---
title: Slow control database 
description: Overview of slow control database and associated systems. 
layout: basic
---

## Slow control database and associated systems

This page provides an overview of the slow-control database and how to start
the relevant docker containers on the [central raid system](Internal-DB.html)
and [cluster raid system](Cluster-DB.html).  In particular, the following
docker containers are used:

* [File-Server]({{ site.url }}/FileServer-Docker) - Provides nginx reverse proxy and file server for CouchDB server.
* [CouchDB Server]({{ site.url }}/CouchDB-Docker) - Provides the CouchDB server.

The links above provide more detailed information about these particular
containers and their associated functionality.

### Overview

The two docker containers must run in parallel since the File-Server provides a
reverse proxy to the CouchDB server.  This means, every access to the
"database" actually proceeds through the File-Server.  This can be visualized
as follows: 

![nginx, File-Server, couchdb communication]({{ site.baseurl }}/static/nginx_overview.png)

### Installation and administration

#### CouchDB Server

The CouchDB Server should be started before the File-Server.  The command to
start this is:

{% highlight bash %}
docker run -d -v /volume1/docker/Databases:/usr/local/var/lib/couchdb \
-v /volume1/docker/CouchDB/Logs:/usr/local/var/log/couchdb            \
-e "ADMINHASH=***this is omitted for security***" \
--name nEDM-CouchDB \
registry.hub.docker.com/mgmarino/couchdb-docker:latest
{% endhighlight %}

The admin hash, which provides the hashed admin password, can be found on the
[wiki]({{ site.fierlingerwiki }}).

#### File-Server

Starting the file server is as follows:
{% highlight bash %}
docker run -d -p 5984:5984 -p 80:80\
  -v /volume1/Measurements:/database_attachments\
  -e "NGX_VIRTUAL_SERVER_2=db.nedm1@/nedm_head/_design/nedm_head/_rewrite"\
  -e "NGX_VIRTUAL_SERVER_1=raid.nedm1@"\
  --name nEDM-FileServer --link nEDM-CouchDB:db registry.hub.docker.com/mgmarino/fileserver-docker:latest
{% endhighlight %}

The two virtual servers (e.g. `NGX_VIRTUAL_SERVER_*`) provide how the nginx
proxy should route requests.  This means that when one gets a different site
when the different sites are opened:

* [http://db.nedm1](http://db.nedm1) - the nEDM interface
* [http://raid.nedm1](http://raid.nedm1) - the CouchDB Futon interface

_Note_: it may be ideal at some point to hide the CouchdB Futon interface, but
having access to this is ideal from a development point of view.
