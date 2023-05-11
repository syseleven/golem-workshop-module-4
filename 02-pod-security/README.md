# Pod Security Contexts

## Preparation

* Before you begin with the actual exercise please make sure to follow these steps to work in your own environment:

  ```shell
  export YOURNAME=<YOURNAME> # <- please replace <YOURNAME> accordingly
  kubectl create ns ${YOURNAME}
  kubectl label namespace ${YOURNAME} golem-workshop=true
  kubectl config set-context --current --namespace=${YOURNAME}
  ```

---

## Deploy

* Apply Deployment and check creating Pods
  
  ```shell
  kubectl apply -f deployment.yaml
  kubectl get pods -w
  ```

## Analyze

* See that container does not startup and find out why
  * Hint: There is more than 1 error...

    ```shell
    kubectl describe pod/security-context-demo-...
    kubectl logs security-context-demo-...
    ```
