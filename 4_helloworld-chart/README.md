# Helm - Install Metrics Server

Definition of helm : 

> Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.
> Charts are easy to create, version, share, and publish — so start using Helm and stop the copy-and-paste.

Then to be able to scale our applications in the cluster we should add metrics-server addon in the cluster. 

Definition : Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

This install container metrics into the kubernetes api.

```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install --set args={--kubelet-insecure-tls} metrics-server metrics-server/metrics-server --namespace kube-system
```

Check the installation of the package.

```
kubectl get pod -n kube-system
```

# Build Docker image

Now we can build the docker image for this hello-world application

```
docker build -t workshop:1 .
```

![docker-build](../images/docker-build.png)

And copy the image to be available for our Kubernetes Kind cluster:

```
> kind load docker-image -n kubernetes-workshop workshop:1
```

# Create basic helm chart

Let's use helm to create an helm chart.

```
helm create workshop
```

This will create a default helm chart with a deployment, hpa, service and values that can be configured

In templates/ you can see kubernetes objects that as templating variables. values.yaml contains values that will be used to render templates. deployment.yaml is for the kubernetes deployment.

Let's reference our newly created docker image :

```
image:
  repository: workshop < name of the image
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "1" < tag of the image
```

# Deploy the application

Let's deploy it !

Deploy the helm chart :

```
helm install -n workshop --create-namespace workshop .
```

![helm-install](../images/helm-install.png)

Let's check if pods are running now :

```
kubectl -n workshop get pods
```

![helm-install](../images/k-get-pods-2.png)

You should see a pod with the status Running.

# Let's expose it

First we need to change the service type to LoadBalancer in values.yaml

```
service:
  type: LoadBalancer
  port: 80
```

deploy it with :

```
helm upgrade -n workshop workshop .
```

Then start a tunnel to minikube to expose the kubernetes service to your local machine : 

```
kubectl -n workshop port-forward service/workshop 8080:80
```

And now visit http://localhost:8080 you should see hello world !

## 6. Release a new version

In `src/server.py` modify the return in main route to : `return "Hello World! Version 2"`

build docker image with a tag named 2:

```
docker build -t workshop:2 .
```

And copy the image to be available for our Kubernetes Kind cluster:

```
> kind load docker-image -n kubernetes-workshop workshop:2
```

Modify the tag in values.yaml 

```
image:
  repository: workshop
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "2" < HERE
```

Deploy the new version with helm :

```
helm upgrade -n workshop workshop .
```

You should see pods doing a rollout one pod is created and another is terminated when the other one is ready :

![deployment-rollout](../images/deployment-rollout.png)

Now when you visit http://localhost you should see :

Hello World! Version 2

## 6. Scale automatically your application

!!! Warrning this part can be hard tell me if you can't do it sorry.

Enable autoscaling in the helm chart modify in values.yaml : 

```
autoscaling:
  enabled: true < HERE false to true
  minReplicas: 1 < min number of replicas 1
  maxReplicas: 100 < max number of replicas 100
  targetCPUUtilizationPercentage: 50 < HERE from 80 to 50 to target average cpu usage 50%
  # targetMemoryUtilizationPercentage: 80
```

Set bounded resource for the container : 

```
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

Deploy : 

```
helm upgrade -n workshop workshop .
```

You should see a new horizontal pods autoscaler object here 
min / max , replicas = current number of replicas set by the hpa

![hpa-1](../images/hpa-1.png)

In a splitted terminal follow number of pods and hpa metrics : 

in one do : 

```
watch kubectl -n workshop get pods
```

in another one do 

```
watch kubectl -n workshop get hpa
```

![hpa](../images/watch-1.png)

After you have recreated a `port-forward`, launch a first load test  : 

```
echo "GET http://localhost:8080/" | vegeta attack -duration=120s -rate 4 -keepalive false | tee results.bin | vegeta report
```

New pod should come

![hpa](../images/k-get-pods-3.png)

if you describe hpa object `kubectl describe -n workshop hpa workshop` you should see : 

```
  Normal   SuccessfulRescale             2m15s              horizontal-pod-autoscaler  New size: 2; reason: cpu resource utilization (percentage of request) above target
```

If we increase rate a new pods should come : 

```
echo "GET http://localhost:80/" | vegeta attack -duration=120s -rate 7 -keepalive false | tee results.bin | vegeta report
```

```
  Normal   SuccessfulRescale             15s                horizontal-pod-autoscaler  New size: 3; reason: cpu resource utilization (percentage of request) above target
```

Then if we stop scale down should happen after 60s.

event in `kubectl describe -n workshop hpa workshop` :

```
  Normal   SuccessfulRescale             8m46s              horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
```

Congrats you deployed you first application and make it scale !

## Cleanup

```
helm uninstall -n workshop workshop 
kind delete cluster -n kubernetes-workshop
```

Thanks for attending to this workshop !
