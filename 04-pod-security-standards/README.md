# Security

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  export YOURNAME=<YOURNAME>
  kubectl create ns ${YOURNAME}
  kubectl config set-context --current --namespace=${YOURNAME}
  ```

* Clone this repository to your working station and change into the directory for the following exercises

---

## Exercise

## Part 1 - Pod and Container Security

**Task:** Deploy a Pod with proper security features.

* First view `secure-pod.yaml` manifest and inspect the security definitions made.

* Deploy the secure pod

  ```shell
  kubectl -n ${YOURNAME} apply -f secure-pod.yaml
  ```

### Verify

* Connect to the Pod and try some "invasive" commands usually requiring higher permissions.

  ```shell
  kubectl -n $YOURNAME exec -it secure-pod -- sh
  $ whoami
  $ id
  $ date -s 23:00:00
  $ touch /etc/test
  $ touch /tmp/test
  $ exit
  ```

---

## Part 2 - Pod Security Standards

* Now establish a baseline for your personal namespace by configuring it to use "PSS"

  ```shell
  kubectl label --overwrite ns ${YOURNAME} \
  pod-security.kubernetes.io/warn=baseline \
  pod-security.kubernetes.io/warn-version=latest
  ```

### Verify

* Show namespace labels

  ```shell
  kubectl get ns $YOURNAME --show-labels
  ```


* First view the `insecure-pod.yaml` manifest and inspect the security differences.

* Now deploy a pod violating "PSS"

  ```shell
  kubectl -n ${YOURNAME} apply -f insecure-pod.yaml
  ```

## Result

  ```shell
  Warning: would violate PodSecurity "baseline:latest":
  non-default capabilities (container "insecure-pod" must not include
  "NET_ADMIN", "SYS_ADMIN", "SYS_TIME" in securityContext.capabilities.add),
  
  privileged (container "insecure-pod" must not set securityContext.privileged=true)
  
  pod/insecure-pod created
  ```

* Important! with a ruleset of `warn` Pods violating a baseline still are created.
* If you whish to deny Pod creation choose `enforce` mode.

---

## Cleanup

* remove PSS from your namespace before you continue

  ```shell
  kubectl label --overwrite ns ${YOURNAME} \
  pod-security.kubernetes.io/warn- \
  pod-security.kubernetes.io/warn-version-
  ```

* remove the Pods

  ```shell
  kubectl -n $YOURNAME delete -f secure-pod.yaml -f insecure-pod.yaml
  ```

---

### Conclusion

Pods and their containers should make use of K8s' on-board security features wherever possible.

Using Pod Security Standards is a good starting point to grant a baseline security ruleset
and to learn about one's Pod security quality.

Decide how restrictive your ruleset should be.
