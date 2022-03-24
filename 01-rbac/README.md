# Role based access control

## Create admin service account with clusterrole binding

```shell
kubectl create serviceaccount simon-admin

kubectl get serviceaccounts simon-admin -o yaml

kubectl describe secret $(kubectl get secret | grep simon-admin | awk '{print $1}')
kubectl get secrets simon-admin-token-XXXXX -o jsonpath="{.data.token}" |base64 --decode

# Alternative: kubectl get secrets $(kubectl get secret | grep simon-admin | awk '{print $1}') -o jsonpath="{.data.token}" |base64 --decode

# Please leave this Token open for later usage

kubectl create -f simon-admin-cluster-role.yaml

kubectl create -f simon-admin-cluster-rolebinding.yaml
```

## Create service account with rolebinding limited to dev namespace

```shell

kubectl create namespace dev

kubectl -n dev create serviceaccount simon-dev

kubectl -n dev get serviceaccounts simon-dev -o yaml

kubectl -n dev get secrets $(kubectl -n dev get secret | grep simon-dev | awk '{print $1}') -o jsonpath="{.data.token}" |base64 --decode

# Please leave this Token open for later usage

kubectl -n dev create -f simon-dev-role.yaml

kubectl -n dev create -f simon-dev-role-binding.yaml
```

* What is the difference between simon-dev-role.yaml and simon-admin-cluster-role.yaml?

```shell
sdiff --suppress-common-lines simon-dev-role.yaml simon-admin-cluster-role.yaml | colordiff

kind: Role                                                    | kind: ClusterRole
  name: simon-dev                                             |   name: simon-admin
  namespace: dev                                              <
  - nodes                                                     |   - nodes 
```

## Add the user simon-admin and the context to your kubeconfig file

```shell
kubectl config set-credentials simon-admin --token='eyJhbGciOiJSUzI1NiIsImtpZCI6IjM1RjNrTXNJdFM2S1IwRGYyZGlzX0NzdXVXZkU4c3BEQVEzckdNa09nLTgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkamFyb3NjaCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJzaW1vbi1hZG1pbi10b2tlbi01bXg5ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJzaW1vbi1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQwNGY2ZWViLTdhZjctNDBmNS1hY2IxLWVlZTQyNzQ0NjRmYyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkamFyb3NjaDpzaW1vbi1hZG1pbiJ9.hkRQeiiUEo-nOxNvx0uZk9dEqjKrWs_5YpCbOJHw4mDzby3CnQLFe7YvGevyVuTHr6FeplCfLAVLwZHFp0HbpwcDJoTQlhPiamePVSJu8B0W3UbG1OXKuDmyDuob25A7o6vOBucLW4TOTxM7sLpo7Vd7Nrkflyi20A371ef7uNzAVBYUWmfcf5zoke8bDQLloMody6ylOe9ReG0k_jRBC-IAhibShBX4yV8_cqKO1xPQ48J2OtYzwin3mMsk22gbLVIUqjrkEzK_9YAamy_o_ut-KV1z6WxjjoRQ41MQQK21qgP7lLSgF6HvBf5jiiDtJT0n5ANeb8tYq2Ln3FJciw'

kubectl config set-context --cluster=nv7j4tdnqm --user=simon-admin simon-admin
```

## Add the user simon-dev and the context to your kubeconfig file

```shell
kubectl config set-credentials simon-dev --token='eyJhbGciOiJSUzI1NiIsImtpZCI6IjM1RjNrTXNJdFM2S1IwRGYyZGlzX0NzdXVXZkU4c3BEQVEzckdNa09nLTgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoic2ltb24tZGV2LXRva2VuLW5sbDZwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InNpbW9uLWRldiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjhkYWVhMWU3LTZkMmMtNGJkNy04MzFmLWM3YzMwY2Q0ODIwYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZXY6c2ltb24tZGV2In0.NgYCan9tF_dmzUXtYHFgdMIJbr3H6yHrMa0MvAQETPCdo1Q2SYSmMtiNg31yc8COXVB2JB5pl5AERcmVEtD6G2QcR_Fe50kDAQrnWgE546TGQrCIvJCLil9bSBS_38q0n2xYY7i7qtdr4vA6nxrlfaRHfDQvqffAYN9S6Vi8YNSmBSvYy5NnG7q3QywoS4DFZ8PFAnvs5YX3QWapzGTHhXrIVT4UUDxHmahe2LFG2Uv4qCgSXZa6KksNfzeLsrqHaNRt-YgVNwjDwFO7xeCeYHgHeCbzBZIu7_uzOkQTh1AVkfgfGi60KgCRoYQQr64RjPnL7f30QO0bX-qzLSKsbQ'

kubectl config set-context --cluster=nv7j4tdnqm --user=simon-dev simon-dev
```

## A context can also include an optional namespace

```shell
kubectl config set-context --cluster=nv7j4tdnqm --user=simon-dev --namespace=dev simon-dev
```

## Switching contexts

### Switch to user simon-dev

```shell
kubectl config use-context simon-dev
```

### Switch to user simon-admin

```shell
kubectl config use-context simon-admin
```

## Install the RBAC-Manager

```shell
helm repo add reactiveops-stable https://charts.reactiveops.com/stable

helm upgrade --install rbac-manager reactiveops-stable/rbac-manager --namespace rbac-manager
```

## Install RBAC Lookup

```shell
brew install reactiveops/tap/rbac-lookup
```

## Cleanup

kubectl delete -f simon-admin-cluster-role.yaml simon-admin-cluster-rolebinding.yaml simon-dev-role.yaml simon-dev-role-binding.yaml
kubectl delete namespace dev
kubectl delete serviceaccount simon-dev simon-admin
