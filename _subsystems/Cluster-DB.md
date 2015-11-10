---
title: Cluster DB
description: Overview of database on the cluster network, which mirrors the internal DB.
layout: basic
order: 3
---

## Cluster DB

This system is a [Synology](https://www.synology.com/) RAID system (specifically RS3614xs), with 12 4-TB
disks.  9 of these disks provide RAID5 (total ~29 TB of space) with 3 disks
acting as hot spares.  The system runs a linux distribution
([DSM](https://www.synology.com/en-global/dsm)) and is generally administrated
either from the command line (e.g. ssh) or via a [web interface](https://10.155.59.88:5185).

### Synchronization

The primary purpose of this machine is to synchronize with the
[Internal DB/Raid system](Internal-DB.html).

#### File synchronization

See [here](Internal-DB.html#file-synchronization).

#### CouchDB replication

Replication of the local DB occurs via a
[tunnel through the optimal system](GatewayMachine.html#ssh-routing).  If a new
system needs to be setup to replicate *all* dbs from the internal DB server,
then one can use the following script:

<script src="https://gist.github.com/mgmarino/9e77e4c57b8ec6133ba9.js"></script>
