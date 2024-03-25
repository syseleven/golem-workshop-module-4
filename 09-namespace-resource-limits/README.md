# Namespace Resource Limits

## Create namespace example

```sh
export YOURNAME=<YOURNAME> # <- please replace <YOURNAME> accordingly
kubectl create namespace ${YOURNAME}-limits
kubectl label namespace ${YOURNAME}-limits golem-workshop=true
```

---

## Deploy Quota and Limits

```sh
kubectl apply -f container-limit-range.yml -n <YOURNAME>-limits
```

Deploy test deployment

```sh
kubectl apply -f test-deployment.yaml --namespace <YOURNAME>-limits
```

See that default requests and limits have been applied

```sh
kubectl describe pod --namespace <YOURNAME>-limits
```

Apply quota

```sh
kubectl apply -f pod-quota.yml -n <YOURNAME>-limits
```

Scale up

```sh
kubectl scale deployment test-deployment --replicas 3 --namespace <YOURNAME>-limits
```

See that only one pod can be created

Create second deployment

```sh
kubectl delete -f test-deployment.yaml -n <YOURNAME>-limits

kubectl run test-app --image=nginxdemos/hello --namespace <YOURNAME>-limits --requests="cpu=2"
```

See that pod can not be created

## Delete limits namespace again

```sh
kubectl delete namespace <YOURNAME>-limits
```
