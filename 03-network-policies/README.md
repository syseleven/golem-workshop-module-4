# network-policies

* Let's create a namespace for our needs and switch into it

```shell
kubectl create namespace <YOURNAME>
kubectl label namespace <YOURNAME> golem-workshop=true
kubectl config set-context --current --namespace="${YOUR_NAMESPACE}"
```

* Apply web-application deployment (and service)

```shell
kubectl apply -f web-application-deployment.yaml
```

* Deny all traffic to web-application labeled pods

```shell
kubectl apply -f deny-all.yaml
```

* Start a busybox container pod in different namespace **in a different shell**

```shell
kubectl create namespace <YOUR_NAME>-policy
kubectl run -n <YOUR_NAME>-policy -it --image busybox network-policy-test -- sh

# In the pod: Try to reach your web-application
wget -O- -q web-application.<YOUR_NAME>
```

* See that it times out

i.e.

```shell
/ # wget -O- web-application.djarosch
Connecting to web-application.djarosch (10.240.26.60:80)
```

* Edit the newly <YOUR_NAME>-policy namespace and add the label `network-policy/web-application: allow`, then deploy the allow-web-application.yaml

```shell
kubectl edit namespace <YOUR_NAME>-policy
kubectl apply -f allow-web-application.yaml
```

i.e.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    ...
    network-policy/web-application: allow
  name: djarosch-policy
...
```

* In your test pod, see that it works again:

```shell
kubectl -n <YOUR_NAME>-policy exec -it network-policy-test -- sh
wget -q -O- web-application.<YOUR_NAME>
```

* Remove the test setup

```shell
kubectl delete -f allow-web-application.yaml && \
kubectl delete -f deny-all.yaml && \
kubectl delete -f web-application-deployment.yaml && \
kubectl delete ns <YOUR_NAME>-policy
```
