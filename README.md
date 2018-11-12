# IMPORTANT NOTE - THIS IS AN EXPERIMENTAL REPO

This operator is developed using the Rook framework.
All official and stable releases of the operator live in the [rook](https://github.com/rook/rook) project.
The purpose of this repository is to push experimental features for users to try out. 
All the development work is also done in the [rook](https://github.com/rook/rook) project.

**DO NOT USE IN PRODUCTION**

# Quickstart

[Cassandra](http://cassandra.apache.org/) is a highly available, fault tolerant, peer-to-peer NoSQL database featuring lightning fast performance and tunable consistency. It provides massive scalability with no single point of failure.

[Scylla](https://www.scylladb.com) is a close-to-the-hardware rewrite of Cassandra in C++. It features a shared nothing architecture that enables true linear scaling and major hardware optimizations that achieve ultra-low latencies and extreme throughput. It is a drop-in replacement for Cassandra and uses the same interfaces, so it is also supported by Rook.

## Prerequisites

A Kubernetes cluster is necessary to run the Rook Cassandra operator.
 * Kubernetes Version must be >= 1.9
 * The user applying the yamls needs to have cluster-admin rbac permissions
 * [PersistentVolumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) are used as storage. If not using a dynamically-provisioned storageClass, the user needs to provision the PersistentVolumes.
 
## Deploy Cassandra Operator

First deploy the Rook Cassandra Operator using the following commands:

```console
cd cluster/examples/kubernetes/cassandra
kubectl apply -f operator.yaml
```

This will install the operator in namespace rook-cassandra-system. You can check if the operator is up and running with:
 
 ```console
  kubectl -n rook-cassandra-system get pod
 ```
 
 ## Create and Initialize a Cassandra/Scylla Cluster
 
 Now that the operator is running, we can create an instance of a Cassandra/Scylla cluster by creating an instance of the `clusters.cassandra.rook.io` resource.
 Some of that resource's values are configurable, so feel free to browse `cluster.yaml` and tweak the settings to your liking.
 Full details for all the configuration options can be found in the [Cassandra Cluster CRD documentation](cassandra-cluster-crd.md).
 
 When you are ready to create a Cassandra cluster, simply run:
 
 ```console
 kubectl create -f cluster.yaml
 ```
 
 We can verify that a Kubernetes object has been created that represents our new Cassandra cluster with the command below.
 This is important because it shows that Rook has successfully extended Kubernetes to make Cassandra clusters a first class citizen in the Kubernetes cloud-native environment.
 
 ```console
 kubectl -n rook-cassandra get clusters.cassandra.rook.io
 ```
 
 To check if all the desired members are running, you should see the same number of entries from the following command as the number of members that was specified in `cluster.yaml`:
 
 ```console
 kubectl -n rook-cassandra get pod -l app=rook-cassandra
 ```
 
 You can also track the state of a Cassandra cluster from its status. To check the current status of a Cluster, as well as events in the cluster, run:
 
 ```console
 kubectl -n rook-cassandra describe clusters.cassandra.rook.io rook-cassandra
 ```
 
 ## Accessing the Database

* From kubectl:

To get a cqlsh shell in your new Cluster:
 ```console
 kubectl exec -n rook-cassandra -it rook-cassandra-east-1-east-1a-0 -- cqlsh
 > DESCRIBE KEYSPACES;
 ```
 
 
* From inside a Pod:

 When you create a new Cluster, Rook automatically creates a Service for the clients to use in order to access the Cluster. The service's name follows the convention `<cluster-name>-client`. You can see this Service in you cluster by running:
 ```console
 kubectl -n rook-cassandra describe service rook-cassandra-client
 ```
 Pods running inside the Kubernetes cluster can use this Service to connect to Cassandra.
 Here's an example using the [Python Driver](https://github.com/datastax/python-driver):
 ```python
 from cassandra.cluster import Cluster
 
 cluster = Cluster(['rook-cassandra-client.rook-cassandra.svc.cluster.local'])
 session = cluster.connect()
```

## Scale Up

The operator supports scale up of a rack as well as addition of new racks. To make the changes, you can use `kubectl edit clusters.cassandra.rook.io rook-cassandra`:
 * To scale up a rack, change the `Spec.Members` field of the rack to the desired value.
 * To add a new rack, append the `racks` list with a new rack.
  
## Scale Down
The operator supports scale down of a rack. To make the changes, you can use `kubectl edit clusters.cassandra.rook.io rook-cassandra`:
* To scale down a rack, change the `Spec.Members` field of the rack to the desired value.

 ## Clean Up
 
To clean up all resources associated with this walk-through, you can run the commands below.

**NOTE:** that this will destroy your database and delete all of its associated data.

```console
kubectl delete -f cluster.yaml
kubectl delete -f operator.yaml
```

## Troubleshooting

If the cluster does not come up, the first step would be to examine the operator's logs:

```console
kubectl -n rook-cassandra-system logs -l app=rook-cassandra-operator
```

If everything looks OK in the operator logs, you can also look in the logs for one of the Cassandra instances:

```console
kubectl -n rook-cassandra logs rook-cassandra-0
```