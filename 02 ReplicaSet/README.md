# ReplicaSet manifest help

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
  Labels is an optional field and can be anything, just tags given to a ReplicaSet. Useful in case of filtering resources to apply rules/actions.

### spec

Contains key information on pods like replicas and selector.
Selector can either have matchLabels or matchExpressions. Either is fine. As a rule of thumb, matchExpressions are more powerful and used in newer resources.

## Using the manifest yaml file

#### Creating a ReplicaSet

```sh
kubectl create -f <file.yaml>
kubectl create -f nginx-rs.yaml
```

#### Verify ReplicaSet status

```sh
kubectl get rs                       # List all the ReplicaSets, including current number of replicas
kubectl get rs nginx-rs              # List a specific ReplicaSet, including current number of replicas

kubectl get rs -o wide               # To display more details like containers and images, use the wide flag

kubectl get rs -l tier=frontend      # Display ReplicaSets only with a specific label

kubectl get rs nginx-rs -o yaml      # Print a specific ReplicaSet's configuration in YAML format
kubectl get pod nginx-pod -o json    # Print a specific ReplicaSet's configuration in JSON format
```

#### Describe ReplicaSet

```sh
kubectl describe rs nginx-rs         # Displays everything about replica set along with events from start to current time
```

#### Testing usecases of ReplicaSet

###### If a pod dies

Nothing needs to be done since ReplicaSet takes care of bringing up a new pod in place of old pod

###### Scale uo or down

```sh
kubectl scale rs nginx-rs --replicas=5
kubectl scale rs nginx-rs --replicas=2
```

#### Delete a ReplicaSet

```sh
kubectl delete rs nginx-rs          # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
kubectl delete -f nginx-rs.yaml     # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
```
