### Exposing Containers
- **kubectl expose** creates a servcie for existing pods
- a service is a stable address for pod(s)
- Using a service is to connect pods
- CordDNS allows us to resolve services by name

# https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
```
Exposing the Service
For some parts of your applications you may want to expose a Service onto an external IP address. Kubernetes supports two ways of doing this: NodePorts and LoadBalancers. The Service created in the last section already used NodePort, so your nginx HTTPS replica is ready to serve traffic on the internet if your node has a public IP.
```

### 4 types of Services
- ClusterIP (default)
    - Single, Internal Vitual IP allocated
    - Only reachable from within cluster (nodes and pods)
    - Pods can reach service on apps port number
- NodePort
    - high port allocated on each node
    - port is open on every node's IP
    - anyone can connect (if they can reach the node)
    - other pods need to be updated to this port
- LoadBalancer
    - Controls a load balance endpoint external to the the cluster
    - only available in some infra providers give you a LB (ex AWS ELB, etc)
    - creates NodePort + ClusterIP services, tells LB to send to NodePort
- ExternalName
    - Adds CNAME DNS record to CoreDNS only
    - Not used for Pods, but for giving pods a DNS name to use for something outside Kubenetes
    
### Cluster IP
- kubectl create deployment httpenv --image=bretfisher/httpenv
    - bretfisher/httpenv is simple apache server spitting back envrionment variables
- kubectl scale deployment/httpenv --replicas=5
- kubectl expose deployment/httpenv --port 8888
- kubectl get pods
```
NAME                       READY   STATUS    RESTARTS   AGE
httpenv-6bf64f7c4f-brxxt   1/1     Running   0          17s
httpenv-6bf64f7c4f-j2dm7   1/1     Running   0          32s
httpenv-6bf64f7c4f-j7t4g   1/1     Running   0          17s
httpenv-6bf64f7c4f-rf9rc   1/1     Running   0          17s
httpenv-6bf64f7c4f-zb6sv   1/1     Running   0          17s
```
- kubectl get services
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
httpenv      ClusterIP   10.152.183.109   <none>        8888/TCP   8s
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP    3h7m
```

- This IP is cluster internal only, how to curl?
    - kubectl run --generator run-pod/v1 tmp-shell --rm -it image bretfisher/netshoot -- bash
    - curl httpenv:8888
    - for linux, don't need to use generator
    ```
    curl 10.152.183.109:8888
    {"HOME":"/root","HOSTNAME":"httpenv-6bf64f7c4f-zb6sv","KUBERNETES_PORT":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP_ADDR":"10.152.183.1","KUBERNETES_PORT_443_TCP_PORT":"443","KUBERNETES_PORT_443_TCP_PROTO":"tcp","KUBERNETES_SERVICE_HOST":"10.152.183.1","KUBERNETES_SERVICE_PORT":"443","KUBERNETES_SERVICE_PORT_HTTPS":"443","PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"}sijoonlee@ubuntu:~$ 

    ```
### Creating a NodePort and LoadBalancer
- kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort    
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
httpenv      ClusterIP   10.152.183.109   <none>        8888/TCP         6m2s
httpenv-np   NodePort    10.152.183.107   <none>        8888:31862/TCP   6s
kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP          3h13m
```
    - note that 8888:31762 has reversed order comparing Docker
    - 8888 : inside of Cluster
    - 31762 : external to cluster
- NodePort service also creates a Cluster IP
- CluterIP/NodePort/LoadBalancer are additive - each one is creaated on the top of the other
    - CluterIP
    - CluterIP - NodePort
    - CluterIP - NodePort - LoadBalancer
- curl localhost:31862 (now you can access localhost)
``` 
{"HOME":"/root","HOSTNAME":"httpenv-6bf64f7c4f-j2dm7","KUBERNETES_PORT":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP":"tcp://10.152.183.1:443","KUBERNETES_PORT_443_TCP_ADDR":"10.152.183.1","KUBERNETES_PORT_443_TCP_PORT":"443","KUBERNETES_PORT_443_TCP_PROTO":"tcp","KUBERNETES_SERVICE_HOST":"10.152.183.1","KUBERNETES_SERVICE_PORT":"443","KUBERNETES_SERVICE_PORT_HTTPS":"443","PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"}    
```
- kubectl expose deployment/httpenv --port 8888 --name httpenv-lb --type LoadBalancer
    
### Kubernetes Service DNS
- curl <hostname> only works in the same namespace
- kubectl get namespaces
- Services also have a FQDN
    - curl <hostname>.<namespace>.svc.cluster.local


### Kubernetes Ingress

