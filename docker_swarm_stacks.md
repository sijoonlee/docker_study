# Stacks: Production-Level Compose
- Stacks accept compose files as their declarative definition for services, networks, and volumes
- command : docker stack deploy
- compose ignores 'deploy' and Swarm ignores 'build'
- Stacks is meant to be used after building up
- docker-compose cli not needed on Swarm server

### Example 1
- under docker_stack_example/swarm_stack_1
docker stack deploy -c exmample-voting-app-stack.yml voteapp
```
Creating network voteapp_backend
Creating network voteapp_default
Creating network voteapp_frontend
Creating service voteapp_redis
Creating service voteapp_db
Creating service voteapp_vote
Creating service voteapp_result
Creating service voteapp_worker
Creating service voteapp_visualizer
```

docker stack ls
```
NAME                SERVICES            ORCHESTRATOR
voteapp             6                   Swarm
```
docker stack services voteapp

docker stack ps voteapp

docker network ls


# Secrets Storage
- Easiest secure solution for storing secrets in Swarm
- Username, password, TLS key, SSH key, and so on
- supports generic strings or binary up to 500kb
- doesn't require apps to be rewritten
- As of Docker 1.13.0 Swarm Raft DB is encrypted on disk
- only stored on disk on manager nodes
- By default, managers and workers "control plane" is TLS + mutual auth
- secrets are stored first in Swarm, then assigned to a service(s)
- only containers in that service(s) can see them
- looks like file, but in-memory fs
- /run/secrets/<secret-name> or /run/secrets/<secret-alias>
- docker-compose can use file-based secret, but not very secure
- version >= 3.1 to use stack and secret

### Example 1
- assuming that psql_username.txt is the secret
- docker create psql_user psql_username.txt
- echo "myDBpassword" | docker secret create psql_pass -
- docker secret ls
- docker secret inspect psql_user
    - can't see the inside of secret itself
- docker service create --name psql \
    --secret psql_user \
    --secret psql_pass \
    -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass \
    -e POSTGRES_USER_FILE=/run/secrets/psql_user postgres 
- docker container run -it <image_id> bash
    - cat /run/secrets/psql_user
    - cat /run/secrets/psql_pass
    - secrets accessible
- docker service update --secret-rm
    - remove secret
    - will redeploy the container
    - can't delete and have new password in the same container (container is immutable)
- need to remove secret(docker secret rm) after deleting service

### Example 2
- see stack-with-build-deploy-secret.yml file under docker_stack_example/swarm-stack-2
- docker stack deploy -c stack-with-build-deploy-secret.yml mydb
- docker secret ls
- docker stack rm mydb
    - will remove secret as well

### Example 3
- see docker-compose.yml under docker_stack_example/compose-secret-assignment
- echo "some_secret_password" | docker secret psql-pw -
- docker stack deploy -c docker-compose.yml drupal
- docker stack ps drupal

### Example 4
# docker-compose.yml
```
version: "3.1"

services:
  psql:
    image: postgres
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt
```
- see docker-compose.yml under docker_stack_example/compose-secret-assignment
- docker-compose up -d
- docker-compsoe exec psql cat /run/secrets/psql_user
    - it's there!!
- under the hood, docker-compose use -v to the secret file in the local hard drive
    - it's not secure
    - it works only with file-based secret
