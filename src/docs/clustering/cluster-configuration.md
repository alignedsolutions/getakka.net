---
layout: docs.hbs
title: Akka.Cluster Configuration
---

# Akka.Cluster Configuration
In this section, we'll review the most important and commonly used Cluster configuration options. To see the full list of options, check out the [full Cluster conf file here](https://github.com/akkadotnet/akka.net/blob/dev/src/core/Akka.Cluster/Configuration/Cluster.conf).

## Critical Configuration Options
Here are the most common options for configuring a [node](./cluster-overview#key-terms) in the cluster:

- **`min-nr-of-members`**: the minimum number of member nodes required to be present in the cluster before the leader marks nodes as `Up`.
    - **`min size per role`**: the minimum number of member nodes for the given role required to be present in the cluster before the role leader marks nodes with the specified role as `Up`.
- **`seed-nodes`**: The addresses of the [seed nodes](./cluster-overview#seed-nodes) to join automatically at startup. Comma-separated list of URI strings. A node may list itself as a seed node and "self-join."
- **`roles`**: the [roles](./cluster-overview#key-terms) that this node is to fulfill in the cluster.  List of strings, e.g. `roles = ["A", "B"]`.

You can see all of these options being used in the following HOCON:

```xml
akka {
   actor.provider = "Akka.Cluster.ClusterActorRefProvider, Akka.Cluster"
    remote {
        helios.tcp {
            port = 8081
            hostname = localhost
        }
    }
    cluster {
       seed-nodes = ["akka.tcp://ClusterSystem@127.0.0.1:8081"] # address of seed node
       roles = ["crawler", "logger"] # roles this member is in
       crawler.min-nr-of-members = 3 # crawler role minimum node count
    }
}
```

## Specifying Minimum Cluster Sizes
One feature of Akka.Cluster that can be useful in a number of scenarios is the ability to specify a minimum cluster size, i.e. "this cluster must have at least 3 nodes present before it can be considered 'up'."

Akka.Cluster supports this behavior at both a cluster-wide and a per-role level.

### Cluster-Wide Minimum Size
If you want to specify a cluster-wide minimum size, then we need to set the `cluster.min-nr-of-members` property inside our HOCON configuration, like this:

```xml
akka {
   actor.provider = "Akka.Cluster.ClusterActorRefProvider, Akka.Cluster"
    remote {
        helios.tcp {
            port = 8081
            hostname = localhost
        }
    }
    cluster {
       seed-nodes = ["akka.tcp://ClusterSystem@127.0.0.1:8081"]
       min-nr-of-members = 3
    }
}
```

In this case, the leader of the cluster will delay firing any `ClusterEvent.MemberUp` messages until 3 nodes have connected to the leader and have not become unreachable. So you can use this setting in combination with your own user-defined actors who subscribe to the `Cluster` for gossip messages (`MemberUp`, specifically) to know if your cluster has reached its minimum size or not.

### Per-Role Minimum Size
If you want to delay nodes of a specific role from being marked as up until a certain minimum has been reached, you can accomplish that via HOCON too!

```xml
akka {
   actor.provider = "Akka.Cluster.ClusterActorRefProvider, Akka.Cluster"
    remote {
        helios.tcp {
            port = 8081
            hostname = localhost
        }
    }
    cluster {
       seed-nodes = ["akka.tcp://ClusterSystem@127.0.0.1:8081"]
       roles = ["crawlerV1", "crawlerV2"]
       crawlerV1.min-nr-of-members = 3
    }
}
```

This tells the role leader for `crawlerV1` to not mark any of those nodes as up until at least three nodes with role `crawlerV1` have joined the cluster.

## Additional Resources
- [`Cluster.conf`](https://github.com/akkadotnet/akka.net/blob/dev/src/core/Akka.Cluster/Configuration/Cluster.conf): the full set of configuration options

### Related Documentation
- [Akka.Cluster Overview](./cluster-overview)
    - [What is a Cluster?](./cluster-overview#what-is-a-cluster-)
    - [Benefits](./cluster-overview#benefits-of-akka-cluster)
    - [Use Cases](./cluster-overview#use-cases)
    - [Terminology](./cluster-overview#key-terms)
    - [Enabling Akka.Cluster](./cluster-overview#enabling-akka-cluster)
    - [Cluster Gossip](./cluster-overview#cluster-gossip)
    - [Nodes](./cluster-overview#nodes)
    - [How a Cluster Forms](./cluster-overview#how-a-cluster-forms)
- [Cluster Routing](./cluster-routing)
    - [How Routers Use Cluster Gossip](./cluster-routing#how-routers-use-cluster-gossip)
    - [Cluster Routing Strategies](./cluster-routing#cluster-routing-strategies)
    - [Types of Clustered Routers](./cluster-routing#types-of-clustered-routers)
    - [Clustered Router Configuration](./cluster-routing#cluster-router-config)
- [Accessing the Cluster `ActorSystem` Extension](./cluster-extension)
    - [Getting a Reference to the `Cluster`](./cluster-extension#getting-a-reference-to-the-cluster-)
    - [Working With Cluster Gossip](./cluster-extension#working-with-cluster-gossip)
    - [Cluster Gossip Event Types](./cluster-extension#cluster-gossip-event-types)
    - [Getting Cluster State](./cluster-extension#getting-cluster-state)