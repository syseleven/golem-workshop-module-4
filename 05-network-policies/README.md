# Network Policies

## Tasks

* Create a web-application
* Apply network policies to deny or allow traffic to the web-application

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  export YOURNAME=<YOURNAME> # <- please replace <YOURNAME> accordingly
  kubectl create ns ${YOURNAME}
  kubectl config set-context --current --namespace=${YOURNAME}
  ```

---

## Exercise

### Deploy the application

* Deploy the web-application

  ```shell
  kubectl apply -f web-application-deployment.yaml
  ```

### Deny traffic to it

* Deny all traffic to web-application labeled pods

  ```shell
  kubectl apply -f deny-all.yaml
  ```

* Start a busybox pod in a different namespace (use a different shell)

  ```shell
  kubectl create namespace ${YOURNAME}-external
  kubectl run -n ${YOURNAME}-external -it --image busybox network-policy-test -- sh
  
  # In the pod: Try to access your web-application
  wget -O- -q web-application.<YOURNAME> # <- replace with your name
  ```

* Notice the request time out

  ```text
  Connecting to web-application.<YOURNAME> (10.240.26.60:80)
  ```

* Exit the busybox pod with `exit`

---

### Allow traffic to it

* Add the label `network-policy/web-application: allow` to your newly created external-namespace

  ```shell
  kubectl label namespace ${YOURNAME}-external network-policy/web-application=allow
  ```

* then deploy a network policy to your personal namespace to allow some external traffic by applying `allow-web-application.yaml`

  ```shell
  kubectl apply -f allow-web-application.yaml
  ```

* In your busybox test pod, see that it works again:

  ```shell
  kubectl -n ${YOURNAME}-external exec -it network-policy-test -- sh
  wget -q -O- web-application.<YOUR_NAME> # <- replace with YOUR NAME accordingly
  ```

* Exit the busybox pod with `exit`

* You have finished this example.

---

## Cleanup (optional)

```shell
kubectl delete -f allow-web-application.yaml && \
kubectl delete -f deny-all.yaml && \
kubectl delete -f web-application-deployment.yaml && \
kubectl delete ns ${YOURNAME}-external
```
