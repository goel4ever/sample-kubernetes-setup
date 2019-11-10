# Storage Volumes

At some point, your app requires storage where the data is stored and accessed. There are various storage options that can be configured in kubernetes. Kubernetes volumes are much more mature than Docker volumes, esp since volumes in k8s are associated with pod, whereas volumes in docker are associated with container. Hence on container restarts volume data is still persisted in case of k8s.
In this section we'll bring an understanding of:

- how storage volumes are handled inside the Kubernetes cluster?
- how data persists beyond pod lifecycle?
- how do containers share data among them?

#### Volumes

Volumes are just a directory containing data that a pod has access to. Volume types are classified into two primary categories:

- Ephemeral volumes - same lifetime as pods
- Durable volumes - beyond pods lifetime

#### Volume Types

There are tons of volume types that are available today. Some of more popular ones are:

- **emptyDir**
  - k8s creates an empty directory first when a pod is assigned to a node
  - Containers inside the pod can read/write to this volume, while the data only stays as long as the pod is running
  - Primary use of this storage is temporary space and share volume across all containers in a specific pod
- **hostPath**
  - Mounts a file or directory from the host node's filesystem into your pod
  - Data remains even after the pod is terminated. Data will be wiped out in case the node itself gets terminated.
  - Closest implementation to docker volume, since even in docker we map host system's directory based on usage
  - Do not use this approach without a very specific requirement, since in case of multiple node environment, data can be out of sync and also have hostPath issues.
- **gcePersistentDisk**
  - Persistent disk on GCE. This volume is mounted into a pod
  - Contents inside are preseved when a pod is mounted/unmounted
  - Volume can be mounted in read-write mode on a single node cluster.
  - Volume can only be mounted in read-only mode on a multi-node clister. Read-Write from multiple sources are forbidden.
  - Restrictions
    - You must create a persistent disk before trying to mount it, which means, the node it is being mounted on in write mode cannot create the disk itself.
    - Nodes on which the pods are running must be GCE VMs
    - Those VMs need to be in the same project and zone as the persistent disk.
- **persistentVolumeClaim**
- **configMap**
- **secret**

## emptyDir

#### Creating a pod

```sh
kubectl create -f pod-ed.yaml
kubectl get po

# Displays everything from the time pod is sent to the node to the current status.
# Look for Mounts and Volumes
kubectl describe pod test-ed
```

#### Verify if cache-volume is mounted

###### Method 1 - Check for existence

```sh
kubectl exec test-ed df /cache          # verify if the volume was mounted or not. And if so, display its details including usage
```

###### Method 2 - Deeper Dive

```sh
kubectl exec -it test-ed -- /bin/bash   # Enter interactive mode in terminal of the pod
cd /cache                               # Navigate to the volume
echo Hello > test-file                  # Create a test file

# List running processes
apt-get update
apt-get install procps
ps aux

kill <pid>                              # Kill the Redis process

# At this point, the Container has terminated and restarted. This is because the Redis Pod has a restartPolicy of Always.
# Once the container is restarted, verify if the test-file still exists
kubectl exec -it test-ed -- /bin/bash
root@test-ed:/data# cd /cache
root@test-ed:/cache# ls
test-file
```

#### Delete a pod

```sh
kubectl delete pod test-ed       # Deletes an existing pod
```

## GCE PersistentDisk

#### Create a persistent disk first

```sh
## Create a disk if not exist already
gcloud compute disks create \
  --size=10GB \
  --zone=us-central1-a \
  my-data-disk

## If you created a new disk manually, it's not used by anyone yet. Hence you'll need format it and then mount it (defined in manifest file)
```

#### Creating a pod

```sh
kubectl create -f pod-pd.yaml
kubectl get po -o wide          # Notice the node on which the pod is running

# Displays everything from the time pod is sent to the node to the current status.
# Look for Mounts and Volumes
kubectl describe pod gce-pd
```

#### Verify if test-pd is mounted

###### Method 1 - Check for existence

```sh
kubectl exec gce-pd df /test-pd          # verify if the volume was mounted or not. And if so, display its details including usage
```

###### Method 2 - Deeper Dive

```sh
kubectl exec -it gce-pd -- /bin/bash    # Enter interactive mode in terminal of the pod
cd /test-pd                             # Navigate to the volume
echo Hello > test-file                  # Create a test file

# Delete the pod and verify its deleted
kubectl delete po gce-pd
kubectl get po

kubectl create -f pod-pd.yaml           # Recreate the pod again

# Once the pod is recreated, verify if the test-file still exists
kubectl exec -it gce-pd -- /bin/bash
root@gce-pd:/data# ls /test-pd
test-file
```

#### Delete a pod

```sh
kubectl delete pod gce-pd              # Deletes an existing pod
```

## configMap

ConfigMap is a kubernetes object which allows you to separate your configuration from pods and components. On a very high level, it is a storage for configuration management, with data stored as key-value pair. ConfigMaps can be created using:

- Directories
- Files
- Literal values

Custom configurations can be applied via:

- Configuration files
- Command line arguments
- Environment variables

> Remember: ConfigMaps are not meant to store any sensitive information like keys, secrets or passwords. We'll use another kubernetes object for that purpose called `secret`.
> ConfigMaps are designed to handle overall configuration of the container.

> Note: You much create a ConfigMap and keys in it before referencing it into a Pod spec, otherwise the pod will fail to start.

#### Create a configMap

```sh
kubectl create configmap <map-name> <data-source>
kubectl create configmap <map-name> --from-file=<path>        # Path to directory/file
kubectl create configmap <map-name> --from-literal=key=value  # Key-Value paid

kubectl create configmap game-config --from-file=./configmaps
kubectl create configmap example-redis-config --from-file=./configmaps/redis-config
kubectl create configmap special-config --from-literal=special.how=very
```

#### Verify the configMap created

```sh
kubectl get configmap game-config         # Lists out number of data and age of the configMap
kubectl get configmap game-config -o yaml # Lists out all the key-value pairs
kubectl describe configmap game-config    # Lists out all the key-value pairs
```

#### Create and verify a pod with configMap set

```sh
kubectl create -f pod-cm-volume.yaml              # Create a pod
kubectl exec -it test-cm -- /bin/sh               # Connect to pod's terminal
root@test-cm:/data# cat /redis-master/redis.conf  # Display config map key-value pairs

kubectl create -f pod-cm-literal.yaml             # Create a pod
kubectl logs test-cm | grep SPECIAL               # Since box is a busybox, we're firing a command to set it as an environment variable
```

#### Delete configMap

```sh
kubectl delete po test-cm                         # Delete an existing pod
kubectl delete configmap game-config              # Delete an existing configMap
kubectl delete configmap example-redis-config     # Delete an existing configMap
kubectl delete configmap special-config           # Delete an existing configMap
```

## secret

A kubernetes object that contains small amount of sensitive data, like password, token, key, etc. On a high level, secrets are used to reduce the risk of exposing confidential information. Stored in etcd datastore.
Secrets are created outside of pods, without caring which pod is going to use it. Secrets are sent only to the target nodes.

> Note: Make sure to create secret before creating the pod
> Remember: Each secret can be at max of 1Mb size

secret can be of one of the following types:

- generic
  - file
  - directory
  - literal value
- docker-registry
- tls

#### Create a secret

###### Create using kubectl

```sh
kubectl create secret <type> <name> <data>
kubectl create secret <type> <name> --from-file=<directory>
kubectl create secret <type> <name> --from-file=<file>
kubectl create secret <type> <name> --from-literal=<key>=<value>

# Create the desired files
echo -n 'admin' > ./username.txt
echo -n '1f2d3g4j54435' > ./password.txt

# Create the secret
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```

###### Create manually

```sh
echo -n 'admin' | base64
echo -n '1f2d3g4j54435' | base64
kubectl create -f secret-db.yaml
```

#### Decoding & Verify the secret

```sh
kubectl get secret db-user-pass         # Display all the secrets exposed based on the command
kubectl describe secret db-user-pass    # Display complete details of a specific secret. Actual value of a secret will not be displayed.

kubectl get secret db-user-pass -o yaml   # Get the encoded secrets from this command

# Display decoded secret values
echo 'YWRtaW4=' | base64 --decode
echo 'MWYyZDF1MmU2N2Rm' | base64 --decode
```

#### Consuming Secrets as Volumes and Env variables

```sh
kubectl create -f pod-secret-volume.yaml
```

#### Verify secret in a pod

###### Secret as a volume

```sh
kubectl exec test-secret ls /etc/foo
kubectl exec test-secret cat /etc/foo/username    # Displays decrypted value
kubectl exec test-secret cat /etc/foo/password    # Displays decrypted value
```

###### Secret as Environment variable

```sh
kubectl exec test-secret-env env | grep SECRET    # Displays decrypted value
```
