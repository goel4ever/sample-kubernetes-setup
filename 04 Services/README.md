# Service manifest help

Handles scenarios like:

- Expose frontend web app to the outside world
- Let frontend connect to the backend app
- Resolve pod ip changes when they die

Types of Services:

- **ClusterIP** - Reachable only within the cluster. Usually backends are configured with ClusterIP. This is what a frontend app would use to connect to backend.
- **NodePort** - Expose app to the outside world
- **LoadBalancer** - Expose app to the outside world. Balance incoming traffic across all the nodes
- **ExternalName** - `GCP doc says this is available`
- **Headless** - `GCP doc says this is available`

## NodePort Service

Let's you expose a service on a NodeIP and a static port (_ranges between 30000 to 32767_) called NodePort. Hence behind the scenes it is masking issues like changing ip addresses, load balancing requests across multiple instances of the apps in the same node or different nodes.

Downsides:

- Only one service per port
- Ports are limited to 30000 and 32767
- You need to deal with Node/VM IP address changes, if it happens

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
  Labels is an optional field and can be anything, just tags given to a service. Useful in case of filtering resources to apply rules/actions.

### spec

Contains key information on selector, service type and ports.
Selector can NOT be either have matchLabels or matchExpressions, but a key value pair

## Using the manifest yaml file

### Setup using NodePort

#### Creating a Service

```sh
kubectl create -f nginx-deploy.yaml     # Make sure to create a deployment before creating a service or use an existing deployment to skip this step
kubectl create -f my-service-np.yaml    # Create a NodePort service
kubectl apply -f my-service-np.yaml     # Create a NodePort service
```

###### [GCP] Create a firewall rule to allow TCP traffic on your node port

```sh
gcloud compute firewall-rules create test-node-port --allow tcp:[NODE_PORT]
gcloud compute firewall-rules create test-node-port --allow tcp:31000
```

### Setup using Load Balancer

When the applicaiton is deployed on multiple nodes, you'll need a Load balancer instead of a NodePort to route your request. Otherwise you'll need to configure all your node external IPs in your domain DNS, which can result in two concerns:

- If a node crashes, a node will be created. With this each time you'll need to update your DNS settings
- You'll have to completely rely on your domain service to balance load of incoming request, with you having no control over the traffic.

Remember, there is a cost associated with each Load Balancer you add to the application. In case you are looking to avoid this cost, look at ingress (example in separate section)

```sh
kubectl create -f nginx-deploy.yaml     # Make sure to create a deployment before creating a service or use an existing deployment to skip this step
kubectl create -f my-service-lb.yaml    # Create a LoadBalancer service
kubectl apply -f my-service-lb.yaml     # Create a LoadBalancer service
kubectl expose deploy nginx-deploy --name=my-service --port=80 --type=LoadBalancer     # Create a LoadBalancer service

You can now simply list the service and hit the external IP to be able to access the application via browser
```

### Verfication, Testing and Cleanup

#### Verify Service status

```sh
kubectl get service                      # List all the Service, including current number of replicas with status
kubectl get service my-service           # List a specific Service, including current number of replicas with status

kubectl get service -o wide              # To display more details like port and selectors, use the wide flag

kubectl get service -l app=nginx-app     # Display Service only with a specific label

kubectl get service my-service -o yaml   # Print a specific Service's configuration in YAML format
kubectl get service my-service -o json   # Print a specific Service's configuration in JSON format
```

#### Describe Service

```sh
kubectl describe service my-service      # Displays everything about service along with events from start to current time
```

#### Testing usecases of Deployment

#### Delete a Service

```sh
kubectl delete svc nginx-service         # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
kubectl delete service nginx-service     # Deletes an existing ReplicaSet along with all the objects it created like pods, etc
```
