# Application Example 01

In this section we'll setup a complete deployment for a `stateless application` to bring entire understanding of Kubernetes together.

## Application Details

Deploying PHP Guestbook app with Redis. Below is the breakdown of apps

- Deploy - (1 pod) Redis DB Master
- Deploy - (3 pod) Redis DB Slaves
- Deploy - (2 pod) Python Frontend web app
- Service (ClusterIP) - Redis Master
- Service (ClusterIP) - Redis Slave
- Service (LoadBalancer) - Frontend

## Commands to create

Make sure to create Deployments before you create Services

```sh
kubectl create -f redis-master-deploy.yaml
kubectl create -f redis-slave-deploy.yaml
kubectl create -f frontend-deploy.yaml

kubectl create -f redis-master-service.yaml
kubectl create -f redis-slave-service.yaml
kubectl create -f frontend-service.yaml
```

## Testing deployment

Since frontend service is of type loadbalancer, use the external IP from the command below and hit it in the browser

```sh
kubectl get service -l tier=frontend
```

## Deleting all resources

```sh
kubectl delete deployment -l app=redis
kubectl delete service -l app=redis
kubectl delete deployment -l app=guestbook
kubectl delete service -l app=guestbook
```

## References

```sh
https://kubernetes.io/docs/tutorials/stateless-application/guestbook/
```
