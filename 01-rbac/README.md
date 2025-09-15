# Role based access control (RBAC)

## Tasks

* Deploy the kubernetes dashboard
* Create service accounts with different permissions
* and test them on the dashboard

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  export YOURNAME=<YOURNAME> # <- please replace <YOURNAME> accordingly
  kubectl create ns ${YOURNAME}
  kubectl config set-context --current --namespace=${YOURNAME}
  ```

* Clone this repository to your working station and change into the directory for this exercise

---

## Prerequisites

### Deploy the kubernetes dashboard

**Important!** This task should only be done by **1 participant** of this course.

* Deploy the dashboard

  ```shell
  # Add kubernetes-dashboard repository
  helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
  # Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
  helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
  ```

* Option 1 - Connect to the kubernetes API via port-forward (we will use this!)

  ```shell
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
  ```

* Then you can visit the website via: [https://localhost:8443/](https://localhost:8443/) (you need to accept the SSL exception as there is no valid cert installed)

* Option 2 - Connect to the kubernetes API via proxy

  ```shell
  kubectl proxy --port=8001
  ```

* Visit URL:
  [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard-kong-proxy:443/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard-kong-proxy:443/proxy/)

---

* Leave your browser window and your shell session with `kubectl port-forward...` open and use a second shell window for the 
  following example 

---

## Exercise

**Important!** The following examples can be performed by **all participants** of this course.

* Also make sure to set this variable in your shell session windows:

  ```shell
  export YOURNAME=<YOURNAME>
  ```

## Example 1 - Service account with namespace permissions

* Create a service account in your namespace

  ```shell
  kubectl -n ${YOURNAME} create serviceaccount ${YOURNAME}-user
  ```

* First edit the role manifest `user-role.yaml` and adjust your name!

* Now apply the manifest to create a role with user permissions in your namespace

* ```shell
  kubectl -n ${YOURNAME} apply -f user-role.yaml
  ```

* Verify what permissions the user role offers

  ```shell
  kubectl describe  -f user-role.yaml 
  ```

* Now connect your service account with that role by creating a rolebinding

  ```shell
  kubectl -n ${YOURNAME} create rolebinding ${YOURNAME}-user-rolebinding --role=${YOURNAME}-user-role --serviceaccount=${YOURNAME}:${YOURNAME}-user
  ```

* Obtain a token for you service account

  ```shell
  kubectl -n ${YOURNAME} create token ${YOURNAME}-user
  ```

* Result is something like `eyJhbGciOiJSUzI1NiIsImtpZCI6ImVW...`

* Copy your token and paste it on [jwt.io](https://jwt.io)
  * Observe the contents of the token

## Access the dashboard with namespace permission

* Access the dashboard by visiting the URL:
  [https://localhost:8443](https://localhost:8443)

* Use your token to log in

* Tasks
  * Browse the dashboard, there should be objects in your namespace like pods, etc.
  * switch to another namespace like kube-system and you should not be able to obtain ressources
  * sign out of the dashboard

* You have finished this example.

---

## Example 2 - Service account with cluster-wide permissions

* Create a service account in your namespace

  ```shell
  kubectl -n ${YOURNAME} create serviceaccount ${YOURNAME}-admin
  ```

* By default there are several roles and clusterroles with different permissions in Kubernetes

* Observe some of these

  ```shell
  kubectl get clusterrole cluster-admin edit view
  kubectl describe clusterrole cluster-admin
  ```

* Notice that a `cluster-admin` is allowed to manage every k8s resource

* Now connect the service account with this clusterrole by creating a clusterrolebinding

  ```shell
  kubectl -n ${YOURNAME} create clusterrolebinding ${YOURNAME}-admin-clusterrolebinding --clusterrole=cluster-admin --serviceaccount=${YOURNAME}:${YOURNAME}-admin
  ```

* Obtain token for the admin serviceaccount

  ```shell
  kubectl -n ${YOURNAME} create token ${YOURNAME}-admin
  ```

* Result is something like `eyAvbFcrOiBDUzI1NiIsIatgZWI6IgUH...`

* Copy your token

## Access the dashboard with cluster-wide permission

* Acces the dashboard by visiting the URL:
  [https://localhost:8443](https://localhost:8443)

* Use your token to log in

* Tasks
  * Browse the dashboard, there should be objects in your namespace like pods, etc.
  * switch to another namespace like kube-system and you should be able to obtain all ressources
  * sign out of the dashboard

* You have finished this example.

---

## Example 3 - Verify permissions with kubectl

### Task: Verify via kubectl if permissions are correctly set

* Basic "command" to verify permissions of service accounts

  ```shell
  kubectl auth can-i get <K8s-RESOURCE> --as=system:serviceaccount:<SERVICEACCOUNT-NAMESPACE>:<SERVICEACCOUNT-NAME> -n <TARGET-NAMESPACE>
  ```

* Example commands

  ```shell
  # test if your own user service account can get pods in your own namespace
  kubectl auth can-i get pods --as=system:serviceaccount:${YOURNAME}:${YOURNAME}-user -n ${YOURNAME}

  # test if your own user service account can get pods in the default namespace 
  kubectl auth can-i get pods --as=system:serviceaccount:${YOURNAME}:${YOURNAME}-user -n default
  
  # test if your own admin service account can delete pods in the kube-system namespace 
  kubectl auth can-i get pods --as=system:serviceaccount:${YOURNAME}:${YOURNAME}-admin -n kube-system
  ```

* Test various combinations of permissions and actions

---

# Cleanup (optional)

```shell
kubectl -n ${YOURNAME} delete serviceaccount ${YOURNAME}-user ${YOURNAME}-admin
kubectl -n ${YOURNAME} delete role ${YOURNAME}-user-role
kubectl -n ${YOURNAME} delete rolebinding ${YOURNAME}-user-rolebinding
kubectl delete clusterrolebinding ${YOURNAME}-admin-clusterrolebinding
helm -n kubernetes-dashboard uninstall kubernetes-dashboard 
```
