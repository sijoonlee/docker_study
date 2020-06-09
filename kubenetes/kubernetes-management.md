
# Run, Create, Expose Genertors
- These commands use helper templates called "generators"
- Every resource in Kubernetes has a specification of "spec"
> kubectl create deployment sample --image nginx --dry-run -o yaml
    - to see what would have happend if the command is actually executed
    - you can use this YAML as a starting point
    
### Example
- kubectl create deployment test --image nginx --dry-run -o yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

```
- kubectl create job test --image nginx --dry-run -o yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: test
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - image: nginx
        name: test
        resources: {}
      restartPolicy: Never
status: {}
```
- kubectl expose deployment/httpenv --port 80 --dry-run -o yaml
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: httpenv
  name: httpenv
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: httpenv
status:
  loadBalancer: {}
```

# Three Management Approaches
- imperative commands: run, expose, scale, edit, create deployment
- imperative objects: create -f file.yml, etc
- declarative objects: apply -f file.yml or dir\, diff

### production
- use declarative objects way





