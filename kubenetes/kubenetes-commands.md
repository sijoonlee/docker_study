# Cheatsheet
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

# Three ways to create pod

### kubectl run
- single pod creation (kubenetes >= 1.18)
- create Deployemnt, ReplicaSet, Pod (kubenetes <= 1.17)
- kubectl run **pod_prefix** --image **Docker_image**

### kubectl create
- create some resources via CLI or YAML

### kubectl apply
- create/update anything via YAML

### kubectl delete
- kubectl delete deployment my-nginx


# Scale up
### kubectl scale deploy/my-nginx --replicas 2
### kubectl scale deployment my-nginx --replicas 2

# Demo
- kubectl run nginx-test --image nginx
- for >= 1.18,  kubectl create deployment nginx-test --image nginx
- kubectl get all
```
NAME                              READY   STATUS              RESTARTS   AGE
pod/nginx-test-57dbc64c59-47m54   0/1     ContainerCreating   0          6s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   67m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-test   0/1     1            0           6s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-test-57dbc64c59   1         1         0       6s
```
- "kubectl run" creates ...
    - Pods
    - ReplicaSet
    - Deployment
    Deployment (  ReplicaSet (  Pod ( Nginx ) + Pod (Nginx) ) )

- kubectl scale deployment nginx-test --replicas 2
- kubectl get pods
- kubectl logs deployment/nginx-test
- kubectl logs deployment/nginx-test --follow --tail 1
- kubectl logs -l run=nginx-test 
    - collect logs from multiple pods with label of "nginx-text"
- * might be good to use 3rd party tool "Stern" for logging
- kubectl describe pod/nginx-test-57dbc64c59
- kubectl describe pods
    - show all pods
- kubectl get pods -w
    - watch command

