# DaemonSet manifest help

DaemonSet is just another controller with specific purpose. A DaemonSet ensures that all or some nodes in a cluster runs a copy of the pod.
What if there was a need to run a monitoring app on each node. This is where DaemonSet shines.
Real life examples:

- Node monitoring daemon: **collectd**
- Log collection daemon: **fluentd**
- Storage daemon: **ceph**

## Manifest file

All manifest file contain main 4 fields:

- apiVersion
- kind
- metadata
- spec

### apiVersion & kind

| Kind                  | apiVersion | Description                                                               |
| --------------------- | :--------: | ------------------------------------------------------------------------- |
| Pod                   |     v1     | Object is part of first stable release of k8s api. Contains core objects. |
| ReplicationController |     v1     | Object is part of first stable release of k8s api. Contains core objects. |
| Service               |     v1     | Object is part of first stable release of k8s api. Contains core objects. |
| ReplicaSet            |  apps/v1   | Most common group related to running apps                                 |
| Deployment            |  apps/v1   | Most common group related to running apps                                 |
| DaemonSet             |  apps/v1   | Most common group related to running apps                                 |
| Job                   |  batch/v1  | Objects related to batch processing, etc                                  |

### metatdata

Consists of two fields:

- name
- labels
  Labels is an optional field and can be anything, just tags given to a deployment. Useful in case of filtering resources to apply rules/actions.

### spec

Contains key information on pods like template and selector.
Selector can either have matchLabels or matchExpressions. Either is fine. As a rule of thumb, matchExpressions are more powerful and used in newer resources.

## Using the manifest yaml file

#### Creating a DaemonSet

```sh
kubectl create -f <file.yaml>
kubectl create -f deploy-daemon.yaml
```

#### Verify Deployment status

```sh
kubectl get ds                       # List all the Deployments, including current number of replicas with status
kubectl get no                       # List a specific Deployment, including current number of replicas with status
kubectl get pod -o wide              # To display more details like containers, images and selectors, use the wide flag
```

#### Describe Deployment

```sh
kubectl describe ds fluent-ds        # Displays everything about deployment along with events from start to current time
```

#### Delete a DaemonSet

```sh
kubectl delete ds fluentd-ds        # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
```
