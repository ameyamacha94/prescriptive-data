# prescriptive-data

# NOTE:

Resources are used are as follows:
1. https://linchpiner.github.io/k8s-multi-container-pods.html
2. https://github.com/nginxinc/docker-nginx/tree/master/stable/alpine

## Steps

1. Build the docker image and push it
2. Edit the kubernetes manifest files
3. Apply the kubernetes manifests
4. See the magic
5. Scale up/down
6. See the magic again

### Building the docker image

1. First please clone the repository
2. Make sure you've docker installed and running
3. Make sure you're able to build and push an image ( as well as pull it)

```
cd docker-nginx
docker build -t <image_name> -f Dockerfile .
docker push <image_name>
```

Please make note of the image_name you use here, as we'll need to use the same in
the k8s-manifests

### Testing on kubernetes

Now, this is the tricky part, where i'm hoping that the following should workout.

So, please launch a kubernetes cluster on your choice of platform/cloud, with
the following in mind:
1. You should be able to access the nodes ( either in browser or through curl )
2. You should be able to access the port that svc exposes on the node(it's 30007),
for this example.
3. You should be able to create a namespace, and also create objects of type
service, configmap and replicaset in that namespace.

So once you've access to a kubernetes cluster as above, then follow next steps:

```
cd k8s-manifests
```

1. Create a namespace with name `nginx-test` as follows: `kubectl create ns nginx-test`
2. Edit the line 27 in `nginx-rs.yaml` and replace <imagename> with the image that
you built in above steps wiht building docker image.
3. Now create the configmap by running `kubectl create -f nginx-cm.yaml`
4. Now create the service by running `kubectl create -f nginx-svc.yaml`
5. Now create the replicaset by running `kubectl create -f nginx-rs.yaml`

Please check to see if pod is up and running by running command:
```
kubectl get pods -n nginx-test
```

Once you see 3 pods in running state, get the ip address of any node as follows:

```
kubectl get nodes -o wide
```

You can either use external ip or internal ip whichever works for you, as long as
you can access it through browser. let's call the ip as <node_ip>

Now go to browser and type <node_ip>:30007, it'll show an output as Hello:<pod_ip>

the pod_ip should be same as you see in the output when ran:

```
kubectl get pods -n nginx-test -o wide
```

Now, you can scale up and down by changing replicas as follows:

```
kubectl -n nginx-test scale --replicas=<replicas_number> rs/mc3
```

You should be able to see the ip address in browser change accordingly.

Note: it might take a few mins to actually reflect the change podip, because
when we use nodeport the balancing is done by the kubeproxy.
