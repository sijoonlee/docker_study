### dry-run
- command
    - kubectl apply -f app.yml --dry-run
    - kubectl apply -f app.yml --server-dry-run
- it does say "created" "changed" or etc
- but it doesn't provide what is actually changed inside

### diff
- command
    - kubectl diff -f app.yml
- will show what's difference

### Labels
- labels goes under metadata
- simple list of key:value for identifying your resource later by selecting, grouping, filtering
- example:
    - tier: frontend
    - app: api
    - env: prod
- filtering example
    - kubectl get pods -l app=nginx
    - kubectl apply -f filename.yml -l app=nginx, app=httpenv

### Label Selectors
- The glue/link telling Services and Deployments which pods are theirs
- Many resources use label selectors to bridge resource dependencies
- Taints and Tolerations also control node placement

### Annotations
- more complex, large, non-identifying info
