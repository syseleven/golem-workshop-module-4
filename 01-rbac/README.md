# Role based access control

## Create admin service account with clusterrole binding

* Set `MYNAME` variable to your name to use in future commands

```shell
MYNAME="myname"
```

* Create admin `serviceaccount`

```shell
kubectl create serviceaccount ${MYNAME}-admin

kubectl get serviceaccounts ${MYNAME}-admin -o yaml
```

* Get token for the serviceaccount

```shell
SECRETNAME=$(kubectl get secret | grep "${MYNAME}-admin" | awk '{print $1}')
kubectl describe secret ${SECRETNAME}
CLUSTERADMINTOKEN=$(kubectl get secrets ${SECRETNAME} -o jsonpath="{.data.token}" | base64 --decode)
```

* Replace occurences of `MYNAME` with your name in files `admin-cluster-role.yaml` and `admin-cluster-rolebinding.yaml`
* Also replace `MYNAMESPACE` in `admin-cluster-rolebinding.yaml` with your namespace name
* create `clusterrole` and `clusterrolebinding`

```shell
kubectl create -f admin-cluster-role.yaml
kubectl create -f admin-cluster-rolebinding.yaml
```

### Test admin service account with clusterrolebinding

* Copy your current kubeconfig file (for safety reasons)

```shell
# for example like this
cp ~/.kube/config ~/.kube/config.BAK
```

* Only when using kubectx
  * Make sure you only have the path for the workshop cluster in your `$KUBECONFIG` environment variable

```shell
export KUBECONFIG=~/.kube/configs/workshopcluster
```

* Use imperative `kubectl` commands to modify your used kubeconfig
  * Add user with right token and add context to use

```shell
kubectl config set-credentials ${MYNAME}-admin --token="${CLUSTERADMINTOKEN}"
kubectl config set-context --cluster=CLUSTERID --user=${MYNAME}-admin ${MYNAME}-admin

# get clusterid by using this command
kubectl config get-clusters
```

* Observe the changes in your kubeconfig

```shell
# look into your kubeconfig file directly
cat ~/.kube/config

# use kubectl command to show config
kubectl config view
```

* Use the context

```shell
kubectl config use-context ${MYNAME}-admin

# Test some commands
kubectl get pods -n MYNAMESPACE
kubectl get pods -n kube-system
kubectl get nodes
```

* Switch back to workshop context

```shell
# find name for the context
kubectl config get-contexts

# switch active context like this
kubectl config use-context admin@k8s-workshopXXX/golem-workshop-XXXXX
```

## Create service account with rolebinding limited to dev namespace

* Create additional namespace

```shell
kubectl create namespace ${MYNAME}-dev
kubectl label namespace ${MYNAME}-dev golem-workshop=true
```

* Create new `serviceaccount`
* Retrieve token for serviceaccount

```shell
kubectl -n ${MYNAME}-dev create serviceaccount ${MYNAME}-dev
kubectl -n ${MYNAME}-dev get serviceaccounts ${MYNAME}-dev -o yaml

DEVELOPERTOKEN=$(kubectl -n ${MYNAME}-dev get secrets $(kubectl -n ${MYNAME}-dev get secret | grep ${MYNAME}-dev | awk '{print $1}') -o jsonpath="{.data.token}" | base64 --decode)
```

* Replace occurences of `MYNAME` with your name in files `dev-role.yaml` and `dev-role-binding.yaml`
* Create developer `role` and `rolebinding`

```shell
kubectl -n ${MYNAME}-dev create -f dev-role.yaml
kubectl -n ${MYNAME}-dev create -f dev-role-binding.yaml
```

* What is the difference between `dev-role.yaml` and `admin-cluster-role.yaml`?

```shell
sdiff --suppress-common-lines dev-role.yaml admin-cluster-role.yaml | colordiff

kind: Role                                                    | kind: ClusterRole
  name: simon-dev                                             |   name: simon-admin
  namespace: dev                                              <
  - nodes                                                     |   - nodes 
```

### Test developer service account with rolebinding

* Copy your current kubeconfig file (for safety reasons)

```shell
# for example like this
cp ~/.kube/config ~/.kube/config.BAK
```

* Only when using kubectx
  * Make sure you only have the path for the workshop cluster in your `$KUBECONFIG` environment variable

```shell
export KUBECONFIG=~/.kube/configs/workshopcluster
```

* Use imperative `kubectl` commands to modify your used kubeconfig
  * Add user with right token and add context to use

```shell
kubectl config set-credentials ${MYNAME}-dev --token="${DEVELOPERTOKEN}"
kubectl config set-context --cluster=CLUSTERID --user=${MYNAME}-dev ${MYNAME}-dev

# get clusterid by using this command
kubectl config get-clusters
```

* Observe the changes in your kubeconfig

```shell
# look into your kubeconfig file directly
cat ~/.kube/config

# use kubectl command to show config
kubectl config view
```

* Use the context

```shell
kubectl config use-context ${MYNAME}-dev

# Test some commands
kubectl get nodes
kubectl get pods -n default
kubectl get pods -n ${MYNAME}-dev
```

* Only works in namespace we configured it for
* Check permissions using `kubectl`

```shell
kubectl auth can-i get pods --as=system:serviceaccount:${MYNAME}-dev:${MYNAME}-dev -n ${MYNAME}-dev
kubectl auth can-i get pods --as=system:serviceaccount:${MYNAME}-dev:${MYNAME}-dev -n default
kubectl auth can-i list nodes --as=system:serviceaccount:${MYNAME}-dev:${MYNAME}-dev
```

* Switch back to workshop context

```shell
# find name for the context
kubectl config get-contexts

# switch active context like this
kubectl config use-context admin@k8s-workshopXXX/golem-workshop-XXXXX
```

---

## Optional content

### Install the RBAC-Manager

* NOTE: Can only be performed once per cluster
* This installs the CRD `rbacdefinitions.rbacmanager.reactiveops.io` and necessary controllers etc. to make RBAC a bit easier
* For more information visit the [documentation](https://rbac-manager.docs.fairwinds.com/)

```shell
helm repo add reactiveops-stable https://charts.reactiveops.com/stable
helm repo update

helm upgrade --install rbac-manager reactiveops-stable/rbac-manager --namespace rbac-manager --create-namespace
```

### Install RBAC Lookup

* This is a local CLI tool to make searching through and visualization of RBAC components more handy

* Install

```shell
brew install reactiveops/tap/rbac-lookup
```

* Example usage

```shell
rbac-lookup ${MYNAME}
```

### Cleanup

```shell
kubectl delete -f simon-admin-cluster-role.yaml -f simon-admin-cluster-rolebinding.yaml -f simon-dev-role.yaml -f simon-dev-role-binding.yaml
kubectl delete namespace dev
kubectl delete serviceaccount simon-dev simon-admin
```
