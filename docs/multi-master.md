---
id: multi-master
title: Using Multiple Masters
sidebar_label:  Multiple Master
---

KeyDB now has support with multiple masters when replicating.  This mode is enabled with the following configuration line:

```
active-replica yes
multi-master yes
```

When KeyDB connects with multiple masters it behaves differently than with traditional replication:
 - Multiple invocations of the replicaof command will result in adding additional masters, not replacing the current one
 - KeyDB will not drop its database when sync'ing with the master
 - KeyDB will merge any reads/writes from the master with its own internal database
 - KeyDB will default to last operation wins

This means that a replica with multiple masters will contain a superset of the data of all its masters.  If two masters have a value with the same key it is undefined which key will be taken.  If a master deletes a key that exists on another master the replica will no longer contain a copy of that key.

## Setting up Multi-Master

Multiple Master setup is very similar to active replica setup, but allows more than one replica node (all masters). It allows you to read and write to all instances which can increase reads under high load, and have your other master nodes ready to go in a failure scenario of one. 

With multi-master setup you make each master a replica of other nodes. This can accept many topologies, you could make different variations of ring topologies or make every master a replica of all other masters. If not all are synced, consider failure scenarios and ensure that one break wont cause others to lose their connections.

An example config file:

## Config File

**Note: When setting up multi-master in a config file, make sure to enable `multi-master` and `active-replica` BEFORE setting your `replicaof` commands, as shown in the examples below, otherwise multi-master replication may not work correctly.**

Instance-A config file:

```
# assuming below parameters were set and IP address of this instance is 10.0.0.2
port 6379
requirepass mypassword123
masterauth mypassword123
# you will need to configure the following
multi-master yes
active-replica yes
replicaof 10.0.0.3 6379
replicaof 10.0.0.4 6379
```

Instance-B config file:

```
# assuming below parameters were set and IP address of this instance is 10.0.0.3
port 6379
requirepass mypassword123
masterauth mypassword123
# you will need to configure the following
multi-master yes
active-replica yes
replicaof 10.0.0.2 6379
replicaof 10.0.0.4 6379
```

Instance-C config file:

```
# assuming below parameters were set and IP address of this instance is 10.0.0.4
port 6379
requirepass mypassword123
masterauth mypassword123
# you will need to configure the following
multi-master yes
active-replica yes
replicaof 10.0.0.2 6379
replicaof 10.0.0.3 6379
```

You can also append to config file `keydb-server --multi-master yes --active-replica yes --replicaof [ipaddress] [port] --replicaof [ipaddress] [port]`


## Removing A Replica Node

During operation you may want to remove a node without having to reload configuration files & restart nodes. This can be done using keydb-cli:

```
keydb-cli> REPLICAOF REMOVE [host] [ip]
```

Note that this command will need to be issued on all nodes within the cluster (we do not yet support automatic propogation accross cluster nodes). 
