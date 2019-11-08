# Pod manifest help

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
  Labels is an optional field and can be anything, just tags given to a pod. Useful in case of filtering resources to apply rules/actions.

### spec

A pod consists of one of more containers. Spec defines configuration of each of those containers.

## Using the manifest yaml file

#### Creating a pod

```sh
kubectl create -f <file.yaml>
kubectl create -f nginx-pod.yaml
```

#### Verify pod status

```sh
kubectl get po
kubectl get pod
kubectl get pods

kubectl get pods -o wide             # To display more details like IP address, use the wide flag

kubectl get pods -l tier=frontend    # Display pods only with a specific label

kubectl get pod nginx-pod -o yaml    # Print a specific pod's configuration in YAML format
kubectl get pod nginx-pod -o json    # Print a specific pod's configuration in JSON format
```

#### Describe pod

```sh
kubectl describe pod nginx-pod       # Displays everything from the time pod is sent to the node to the current status
```

#### Testing pod

```sh
ping 10.36.4.3       # Execute the command from master node. This is one way to check connection.

kubectl exec -it nginx-pod -- /bin/sh # Getting a shell of a running pod
# hostname
# exit
```

#### Delete a pod

```sh
kubectl delete pod nginx-pod       # Deletes an existing pod
```
