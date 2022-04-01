# Pod Security Contexts

* Create your personal Namespace

```shell
kubectl create namespace <YOURNAME>
kubectl label namespace <YOURNAME> golem-workshop=true
```

* Set default namespace to context

```shell
kubectl config set-context --current --namespace=<YOUR_NAME>
```

* Apply Deployment and check creating Pods
  
```shell
kubectl apply -f deployment.yaml
kubectl get pods -w # watch kubectl get pod -o wide
```

* See that container does not startup and find out why, fix error

```shell
kubectl describe pod/security-context-demo-...
kubectl logs security-context-demo-...
```
