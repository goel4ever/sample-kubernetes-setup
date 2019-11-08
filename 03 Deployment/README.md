# Deployment manifest help

Deployment is a controller just like ReplicationController or ReplicaSet, but with more features. Moving forward we won't need to handle pods or ReplicaSets, since all of it is taken cared by deployment controller. Deployment is all about updates and rollbacks.
If number of replicas are not mentioned, deployment will create a default replicas with count of 1.
Handles scenarios like:

- Upgrading with zero downtime
- upgrade sequentially, one after the other
- Pause and resume upgrade process
- Rollback upgrade to previous stable release

## Deployment Strategies

- **Recreate** - has downtime when switching from application version A to version B
- **RollingUpdate** - will slowly roll-out the version of the app, one after another. This is the `default` deployment strategy. This update is slower though.
- **Canary** - grdually shifts traffic from application version A to version B. The advantage here is that after updating few servers, you'll have the opportunity to test the application and decide whether to update rest of the servers as well or rollback those few servers back to version A
- **Blue/Green** - with this approach application version B (Green) is dpeloyed with exact same amount of instances as of version A (Blue). Once testing of new version is completed, traffic is switched from version A to version B at `load balancer` level. Advantage of this approach is instant rollout and rollback

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

Contains key information on pods like replicas and selector.
Selector can either have matchLabels or matchExpressions. Either is fine. As a rule of thumb, matchExpressions are more powerful and used in newer resources.

## Using the manifest yaml file

#### Creating a Deployment

```sh
kubectl create -f <file.yaml>
kubectl create -f nginx-deploy.yaml
```

#### Verify Deployment status

```sh
kubectl get deploy                   # List all the Deployments, including current number of replicas with status
kubectl get deploy nginx-deploy      # List a specific Deployment, including current number of replicas with status

kubectl get deploy -o wide           # To display more details like containers, images and selectors, use the wide flag

kubectl get deploy -l app=nginx-app  # Display Deployment only with a specific label
kubectl get rs -l app=nginx-app      # Display ReplicaSet created by the deployment, with a specific label
kubectl get po -l app=nginx-app      # Display Pods created by the deployment, with a specific label

kubectl get deploy nginx-deploy -o yaml   # Print a specific Deployment's configuration in YAML format
kubectl get deploy nginx-deploy -o json   # Print a specific Deployment's configuration in JSON format
```

#### Describe Deployment

```sh
kubectl describe deploy nginx-deploy      # Displays everything about deployment along with events from start to current time
```

#### Testing usecases of Deployment

###### If a pod dies

Nothing needs to be done since deploy takes care of bringing up a new pod in place of old pod

###### Scale uo or down

```sh
kubectl scale deploy nginx-deploy --replicas=5
kubectl scale deploy nginx-deploy --replicas=2
```

#### Delete a Deployment

```sh
kubectl delete deploy nginx-deploy      # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
kubectl delete -f nginx-deploy.yaml     # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
```

### Deployment Strategies

#### Update a deployment

###### Method 1

```sh
kubectl set image deploy nginx-deploy nginx-container=nginx:1.9.1
kubectl set image deploy nginx-deploy nginx-container=nginx:1.9.1 --record  # Records the command being used in history
```

###### Method 2

This method opens up vi editor to make changes in the configuration file. Configuration changes gets applied as soon as we close and save the file in editor.

```sh
kubectl edit deploy nginx-deploy
```

###### Check status of deployment update

```sh
kubectl rollout status deployment/nginx-deploy
```

#### Rollback a deployment

Lookup the history first

```sh
kubectl rollout history deployment/nginx-deploy   # Returns the revisions created. Displays commands executed if --record flag was used
kubectl rollout undo deployment/nginx-deploy
```
