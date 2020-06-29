# Full App Lifecycle with compse
- single set of compose files for various env
- local development environment - docker-compose up
- remote CI environment - docker-compose up
- remote production environment - docker stack deploy
- note that 'compose extends' doesn't work in Stacks yet

### Example
- see docker_stack_example/swarm-stack-3
    ```
    docker-compose.override.yml   
    docker-compose.prod.yml  
    docker-compose.test.yml  
    docker-compose.yml  
    Dockerfile  
    psql-fake-password.txt    
    ```
- See docker-stack-example/swarm-stack-3
- docker-compose.yml
    - this is the base default setting!
- docker-compose.override.yml
    - will be picked up automatically by docker-compose up, on the top of docker-compose.yml
    - docker-compose up
- docker-compose.test.yml
    - docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
- docker-compose.prod.yml
    - docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml
    - "config" will push .yml files together into a single Compose file equivalent

# Service Updates
- Provides rolling replacement of tasks/containers in a service
- Limits downtime (be careful with "prevents" downtime)
- will replace containers for most changes
- has many cli options
    - includes rollback / healthcheck /scale 

### Service Update Examples
- docker service update --image myapp:1.2.1 <servicename>
    - just update
- docker service update --env-add NODE_ENV=production --publish-rm 8080
    - add an environment variable / remove port
- docker service scale web=9 api=6
    - change the number of replicas of two services

### Swarm Updates in Stack Files
- same command, just edit yaml file
- docker stack deploy -c file.yml <stackname>

### Example Demo
- docker service create -p 8088:80 --name web nginx:1.13.7
- docker service scale web=5
- docker service update --image nginx:1.13.6 web
    - change image
- docker service update --publish-rm 8088 --publish-add 9090:80
    - remove port 8088
    - add port 9090
- docker service update --force web
    - docker swarm doesn't rebalancing task over nodes
    - but when you update (even without changing anything), nodes are rebalanced (in terms of resources)
    - those nodes doing nothing will work after updating
- docker service rm web
    - clean-up

### Healthchecks
- Supported in Dockerfile, Compose yaml, docker run, swarm services
- docker engine will exec's the comand in the container
- it expects exit 0(ok) or exit 1(err)
- every 30 seconds(by default), it checks exit 0 or exit 1
    - when it gets exit 1, it marks it as unhealthy container
- healthcheck status shows up in **docker container ls**
- check last 5 healthchecks with **docker container inspect**
- docker run does nothing with healthcheck
- services will replace tasks if they fail helathcheck
- service updates wait for them before continuing
- Example
    ```
    docker run \
    --health-cmd="curl -f localhost:9200/_cluster/health || false" \
    --health-interval=5s \
    --health-retries=3 \
    --health-timeout=2s \
    --health-start-period=15s \
    elasticsearch:2

    ```
    - 9200 is the port **inside** of container, which is used by elasticsearch
    ```
    docker service create --name p1 --health-cmd="pg_isready -U postgrest || exit 1" postgres
    ```
- options
    ```
    --interval=DURATION (default 30s)
    --timeout=DURATION (default 30s)
    --start-period=DURATION (default 0s)
    --retries=N (default 3)
    ```
- Basic command using default options
    ```
    HEALTHCHECK curl -f http://localhost/ || false
    ```
    - both false and exit 1 will work
- Custom options with the command
    ```
    FROM nginx:1.13
    HEALTHCHECK --timeout=2s --interval=3s --retries=3 \
        CMD curl -f http://localhost/ || exit 1
    ```
    ```
    FROM nginx-php-combo-app-image
    HEALTHCHECK --interval=5s --timeout=3s \
        CMD curl -f http://localhost/ping || exit 1
    ```
    - you can use built-in health check address (http://localhost/ping)
- HealthCheck in Compose/Stack File

    ```
    version: "2.1" # minimum version to use healthcheck
    services:
        web:
            image:nginx
            healthcheck:
                test: ["CMD", "curl", "-f", "http://localhost"]
                interval: 1m30s
                timeout: 10s
                retries: 3
                start_period: 1m # supported since version 3.4
    ```