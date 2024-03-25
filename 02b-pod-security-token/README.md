# Pod Security - ServiceAccount Token

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  export YOURNAME=<YOURNAME> # <- please replace <YOURNAME> accordingly
  kubectl create ns ${YOURNAME}
  kubectl config set-context --current --namespace=${YOURNAME}
  ```

---

## Inspect

* Find the default service account in your namespace.

  ```shell
  kubectl -n ${YOURNAME} get serviceaccount
  
  # Result: "default"
  ```

## Deploy a test pod

* Apply Deployment and check creating Pods
  
  ```shell
  kubectl apply -f token-test-pod.yaml
  ```

## Inspect default token

* Inspect the auto-mount of the default service account credentials by jumping into the pod

    ```shell
    kubectl exec -it token-test-pod -- bash
    ```

* Inspect the contents of the mount path

  ```shell
  df -h
  ls /run/secrets/kubernetes.io/serviceaccount
  cat /run/secrets/kubernetes.io/serviceaccount/ca.crt
  cat /run/secrets/kubernetes.io/serviceaccount/namespace
  cat /run/secrets/kubernetes.io/serviceaccount/token
  ```

* Inspect the token contents in https://jwt.io

* Use the gathered pieces of information to query the kubernetes API from within the pod

  ```shell
  env | grep KUBERNETES
  
  APISERVER=${KUBERNETES_SERVICE_HOST}
  SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
  NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)
  TOKEN=$(cat ${SERVICEACCOUNT}/token)
  CACERT=${SERVICEACCOUNT}/ca.crt
  curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET https://${APISERVER}/api
  
  # afterwards exit the pod
  exit
  ```

* Result:
  * It is possible to use the auto-mounted token of the default service account to query the k8s API.
  * Fortunately the service account has almost no privileges...
  * but "ward off the beginnings", what if someone assigned higher privileges to the service account...

## General recommendation

* Why would a pod need access to the k8s API? (Operators, higher functionality, etc.)
* If this feature is not required, it should be deactivated by default
* Ff it is required, use a custom service account and not the default one

---

## Concept of "least privileges"

* Reduce the attack surface by opt-out of the service account token auto-mount

* Edit the pod definition again `token-test-pod.yaml` and set the option `automountServiceAccountToken` to `false`
* Then delete and recreate the pod

  ```shell
  kubectl delete -f token-test-pod.yaml
  kubectl apply -f token-test-pod.yaml
  ```

* Verify the default token is not mounted anymore

  ```shell
  kubectl describe -f token-test-pod.yaml
  kubectl exec -it token-test-pod -- bash
  df -h
  ```

* Result

The mount-path does not exist anymore.

---

## Cleanup (optional)

* Delete the pod again

  ```shell
  kubectl delete -f token-test-pod.yaml
  ```
