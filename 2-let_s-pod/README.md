# kubernetes-workshop

## A new namespace

To your `workshop` namespace : 

```
kubectl create namespace workshop
```

## Let's create a pod

```
> kubectl --namespace workshop apply -f pod.yaml
```

And verify it's health :

```
> kubectl --namespace workshop get pod
```

You can also gather more information of your pod via:

```
> kubectl --namespace workshop describe pod k8s-workshop-nginx-test
```

And test the connectivy of your pod :

```
> kubectl --namespace workshop port-forward pods/k8s-workshop-nginx-test 8080:80
> curl -I http://127.0.0.1:8080/
```

In the end, you can delete the pod:

```
kubectl --namespace workshop delete pod k8s-workshop-nginx-test
```