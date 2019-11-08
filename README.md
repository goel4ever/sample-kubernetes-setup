# Sample Kubernetes Setup

Instructions and sample configuration files for Kubernetes environment

### Quick Tip

#### Test your internal IPs by creating Busybox

To get a prompt of a busybox running inside the network, execute the following command. (A tip is to use one unique container per developer.)

```sh
kubectl run curl-<YOUR NAME> --image=radial/busyboxplus:curl -i --tty --rm
```

You may omit the `--rm` and keep the instance running for later re-usage.
To reuse it later, type:

```sh
kubectl attach <POD ID> -c curl-<YOUR NAME> -i -t
```

###### Note:

Using the command `kubectl get pods` you can see all running POD's. The is something similar to curl-yourname-752648536-kns95.
Note that you need to login to google cloud from your terminal (once) before you can do this!
Here is an example, make sure to put in your zone, cluster and project:

```sh
gcloud container clusters get-credentials example-cluster --zone us-central1-a --project example-183355
```

### Contribute

Please free to throw in suggesstions, point out issues or collaborate with me. I am simply a message away
