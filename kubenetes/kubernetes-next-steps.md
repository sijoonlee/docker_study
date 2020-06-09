# Storage in kubernetes
- StatefulSets is new resource type, making Pods more sticky
- but keep it simple, avoid stateful workloads
- use db-as-a-service if possible

# Volumes in kubernetes
- two types
    - Volumes
        - tied to lifecycle of a pod
        - all containers in a single Pod can share them
    - PersistentVolumes
        - created at the cluster level, outlives a Pod
        - separates storage config from Pod using it
        - multiple pods can share them
- CSI plugins are the new way to connect to stroage
    - Container Storage Interface

# Ingress
- it is to route outside conntections based on hostname or URL
- nginx is popular, but Traefik, HAProxy, F5, Envoy, Istio, etc

# CRD's and The Operator Pattern
- you can add 3rd party resources and controllers
- this extends Kubernetes API and CLI
- a pattern is starting to emerge
- Operator: automate deployment and management of complex apps

# Higher Deployment Abstractions
- all kubectl commands talk to Kubernetes API
- Kubernetes has limited built-in templating, versioning, tracking, and managing of your apps
- Opinionated 3rd party apps are emergin
- "Helm" is the most popular
- "Compose on Kubernetes" comes with Desktop

# Templating YAML
- Helm was the first winner
- Kustomize feature works out-of-box (>= 1.14)
- find related topic "docker app"

# Kubernetes Dashboard
- GUI

# Kubectl Namespaces and Context
- Namespaces limit scope, aka 'virtual clusters'
- kubectl get namespaces
- kubectl get all --all-namespaces
    - will show built-in hidden system stuffs
- Context changes kubectl cluster and namespace
    - see ~/.kube/config
    - kubectl config get-contexts
    - kubectl config set*