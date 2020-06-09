# kubectl apply
- full declarative objects
- kubectl apply -f filename.yml

### usage cases
- create/update resources from a file
    - kubectl apply -f filename.yml
- create/update a whole directory of yml
    - kubectl apply -f yamlfiles/
- create/update from an URL
    - kubectl apply -f https://abc.com/file.yml

### kubernetes objects
- https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

### kubernetes configuration YAML
- each file contains one or more manifests
- each manifest describes an API object (deployment, job, secret)
- each manifest needs four parts (root key:values in the file)
    - apiVersion:
    - kind:
    - metadata:
    - spec:

### Example
- pod.xml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.3
    ports:
    - containerPort: 80
```

- deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3
        ports:
        - containerPort: 80

```

- app.yml
```
apiVersion: v1
kind: Service
metadata:
  name: app-nginx-service
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    app: app-nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-nginx
  template:
    metadata:
      labels:
        app: app-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3
        ports:
        - containerPort: 80
```

# Building YAML files
- kind: refer api-resources
- apiVersion: refer api-versions
- metadata: only name is required
- spec: all actions here

### get all keys each kind supports
- kubectl explain services --recursive
```
KIND:     Service
VERSION:  v1

DESCRIPTION:
     Service is a named abstraction of software service (for example, mysql)
     consisting of local port (for example 3306) that the proxy listens on, and
     the selector that determines which pods will answer requests sent through
     the proxy.

FIELDS:
   apiVersion	<string>
   kind	<string>
   metadata	<Object>
      annotations	<map[string]string>
      clusterName	<string>
      creationTimestamp	<string>
      deletionGracePeriodSeconds	<integer>
      deletionTimestamp	<string>
      finalizers	<[]string>
      generateName	<string>
      generation	<integer>
      labels	<map[string]string>
      managedFields	<[]Object>
         apiVersion	<string>
         fieldsType	<string>
         fieldsV1	<map[string]>
         manager	<string>
         operation	<string>
         time	<string>
      name	<string>
      namespace	<string>
      ownerReferences	<[]Object>
         apiVersion	<string>
         blockOwnerDeletion	<boolean>
         controller	<boolean>
         kind	<string>
         name	<string>
         uid	<string>
      resourceVersion	<string>
      selfLink	<string>
      uid	<string>
   spec	<Object>
      clusterIP	<string>
      externalIPs	<[]string>
      externalName	<string>
      externalTrafficPolicy	<string>
      healthCheckNodePort	<integer>
      ipFamily	<string>
      loadBalancerIP	<string>
      loadBalancerSourceRanges	<[]string>
      ports	<[]Object>
         name	<string>
         nodePort	<integer>
         port	<integer>
         protocol	<string>
         targetPort	<string>
      publishNotReadyAddresses	<boolean>
      selector	<map[string]string>
      sessionAffinity	<string>
      sessionAffinityConfig	<Object>
         clientIP	<Object>
            timeoutSeconds	<integer>
      type	<string>
   status	<Object>
      loadBalancer	<Object>
         ingress	<[]Object>
            hostname	<string>
            ip	<string>
```

### kubectl explain services.spec
```
KIND:     Service
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Spec defines the behavior of a service.
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     ServiceSpec describes the attributes that a user creates on a service.

FIELDS:
   clusterIP	<string>
     clusterIP is the IP address of the service and is usually assigned randomly
     by the master. If an address is specified manually and is not in use by
     others, it will be allocated to the service; otherwise, creation of the
     service will fail. This field can not be changed through updates. Valid
     values are "None", empty string (""), or a valid IP address. "None" can be
     specified for headless services when proxying is not required. Only applies
     to types ClusterIP, NodePort, and LoadBalancer. Ignored if type is
     ExternalName. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   externalIPs	<[]string>
     externalIPs is a list of IP addresses for which nodes in the cluster will
     also accept traffic for this service. These IPs are not managed by
     Kubernetes. The user is responsible for ensuring that traffic arrives at a
     node with this IP. A common example is external load-balancers that are not
     part of the Kubernetes system.

   externalName	<string>
     externalName is the external reference that kubedns or equivalent will
     return as a CNAME record for this service. No proxying will be involved.
     Must be a valid RFC-1123 hostname (https://tools.ietf.org/html/rfc1123) and
     requires Type to be ExternalName.

   externalTrafficPolicy	<string>
     externalTrafficPolicy denotes if this Service desires to route external
     traffic to node-local or cluster-wide endpoints. "Local" preserves the
     client source IP and avoids a second hop for LoadBalancer and Nodeport type
     services, but risks potentially imbalanced traffic spreading. "Cluster"
     obscures the client source IP and may cause a second hop to another node,
     but should have good overall load-spreading.

   healthCheckNodePort	<integer>
     healthCheckNodePort specifies the healthcheck nodePort for the service. If
     not specified, HealthCheckNodePort is created by the service api backend
     with the allocated nodePort. Will use user-specified nodePort value if
     specified by the client. Only effects when Type is set to LoadBalancer and
     ExternalTrafficPolicy is set to Local.

   ipFamily	<string>
     ipFamily specifies whether this Service has a preference for a particular
     IP family (e.g. IPv4 vs. IPv6). If a specific IP family is requested, the
     clusterIP field will be allocated from that family, if it is available in
     the cluster. If no IP family is requested, the cluster's primary IP family
     will be used. Other IP fields (loadBalancerIP, loadBalancerSourceRanges,
     externalIPs) and controllers which allocate external load-balancers should
     use the same IP family. Endpoints for this Service will be of this family.
     This field is immutable after creation. Assigning a ServiceIPFamily not
     available in the cluster (e.g. IPv6 in IPv4 only cluster) is an error
     condition and will fail during clusterIP assignment.

   loadBalancerIP	<string>
     Only applies to Service Type: LoadBalancer LoadBalancer will get created
     with the IP specified in this field. This feature depends on whether the
     underlying cloud-provider supports specifying the loadBalancerIP when a
     load balancer is created. This field will be ignored if the cloud-provider
     does not support the feature.

   loadBalancerSourceRanges	<[]string>
     If specified and supported by the platform, this will restrict traffic
     through the cloud-provider load-balancer will be restricted to the
     specified client IPs. This field will be ignored if the cloud-provider does
     not support the feature." More info:
     https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/

   ports	<[]Object>
     The list of ports that are exposed by this service. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   publishNotReadyAddresses	<boolean>
     publishNotReadyAddresses, when set to true, indicates that DNS
     implementations must publish the notReadyAddresses of subsets for the
     Endpoints associated with the Service. The default value is false. The
     primary use case for setting this field is to use a StatefulSet's Headless
     Service to propagate SRV records for its Pods without respect to their
     readiness for purpose of peer discovery.

   selector	<map[string]string>
     Route service traffic to pods with label keys and values matching this
     selector. If empty or not present, the service is assumed to have an
     external process managing its endpoints, which Kubernetes will not modify.
     Only applies to types ClusterIP, NodePort, and LoadBalancer. Ignored if
     type is ExternalName. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/

   sessionAffinity	<string>
     Supports "ClientIP" and "None". Used to maintain session affinity. Enable
     client IP based session affinity. Must be ClientIP or None. Defaults to
     None. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies

   sessionAffinityConfig	<Object>
     sessionAffinityConfig contains the configurations of session affinity.

   type	<string>
     type determines how the Service is exposed. Defaults to ClusterIP. Valid
     options are ExternalName, ClusterIP, NodePort, and LoadBalancer.
     "ExternalName" maps to the specified externalName. "ClusterIP" allocates a
     cluster-internal IP address for load-balancing to endpoints. Endpoints are
     determined by the selector or if that is not specified, by manual
     construction of an Endpoints object. If clusterIP is "None", no virtual IP
     is allocated and the endpoints are published as a set of endpoints rather
     than a stable IP. "NodePort" builds on ClusterIP and allocates a port on
     every node which routes to the clusterIP. "LoadBalancer" builds on NodePort
     and creates an external load-balancer (if supported in the current cloud)
     which routes to the clusterIP. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
```
### kubectl explain services.spec.type
```
KIND:     Service
VERSION:  v1

FIELD:    type <string>

DESCRIPTION:
     type determines how the Service is exposed. Defaults to ClusterIP. Valid
     options are ExternalName, ClusterIP, NodePort, and LoadBalancer.
     "ExternalName" maps to the specified externalName. "ClusterIP" allocates a
     cluster-internal IP address for load-balancing to endpoints. Endpoints are
     determined by the selector or if that is not specified, by manual
     construction of an Endpoints object. If clusterIP is "None", no virtual IP
     is allocated and the endpoints are published as a set of endpoints rather
     than a stable IP. "NodePort" builds on ClusterIP and allocates a port on
     every node which routes to the clusterIP. "LoadBalancer" builds on NodePort
     and creates an external load-balancer (if supported in the current cloud)
     which routes to the clusterIP. More info:
     https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
```

### kubectl explain deployment.spec.template.spec.volumes.nfs.server
```
KIND:     Deployment
VERSION:  apps/v1

FIELD:    server <string>

DESCRIPTION:
     Server is the hostname or IP address of the NFS server. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#nfs
```

### api resources
- kubectll api-resources
```
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
leases                                         coordination.k8s.io            true         Lease
eniconfigs                                     crd.k8s.amazonaws.com          false        ENIConfig
events                            ev           events.k8s.io                  true         Event
ingresses                         ing          extensions                     true         Ingress
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```

- kubectl api-version
```
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
crd.k8s.amazonaws.com/v1alpha1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```