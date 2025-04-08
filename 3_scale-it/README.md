# kubernetes-workshop

## Let's create a pod

```
> kubectl --namespace workshop apply -f deployment.yaml
```

And verify it's health :

```
> kubectl --namespace workshop get pod
```

## Create a related Service

```
> kubectl --namespace workshop apply -f service.yaml
```

And verify it's health :

```
> kubectl --namespace workshop get service
```

And test the connectivy of your pod :

```
> kubectl --namespace workshop port-forward service/k8s-workshop-nginx-test 8080:80
> curl -I http://127.0.0.1:8080/
```

In the end, you can delete the namespace:

```
kubectl delete namespace workshop
```