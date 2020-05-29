# Docker compose
- configure relationships between containers
- save docker run setting
- consists of
    - YAML-formatted file
        - containers
        - networks
        - volumes
    - CLI tool using YAML file

- docker-compose.yml template
```
    version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

    services:  # containers. same as docker run
    servicename: # a friendly name. this is also DNS name inside network
        image: # Optional if you use build:
        command: # Optional, replace the default CMD specified by the image
        environment: # Optional, same as -e in docker run
        volumes: # Optional, same as -v in docker run
    servicename2:

    volumes: # Optional, same as docker volume create

    networks: # Optional, same as docker network create
```

### install
1. install  
    > sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

2. set permission  
    > sudo chmod +x /usr/local/bin/docker-compose

3. create symlink   
    > sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

4. 
https://docs.docker.com/engine/install/linux-postinstall/
### reference  
https://docs.docker.com/compose/compose-file/#volumes

### commands
- docker-compose up : setup volumes/networks and start all contatiners, build if not found in cache
- docker-compose build : rebuild
- docker-compose down : stop all containers and remove container/volume/networks
- docker-compose ps : show containers
- docker-compose top : show processes in the containers
- docker-compose down --rmi local : delete 
    ```
    --rmi type 
        Remove images. Type must be one of:
        'all': Remove all images used by any service.
        'local': Remove only images that don't have a
        custom tag set by the `image` field.
    ```

# Ports vs Expose
https://stackoverflow.com/questions/40801772/what-is-the-difference-between-docker-compose-ports-vs-expose

### Ports
Expose ports. Either specify both ports (HOST:CONTAINER), or just the container port (a random host port will be chosen).

Ports mentioned in docker-compose.yml will be shared among different services started by the docker-compose.
Ports will be exposed to the host machine to a random port or a given port.
My docker-compose.yml looks like:

mysql:
  image: mysql:5.7
  ports:
    - "3306"
If I do docker-compose ps, it will look like:

  |Name|Command|State|Ports|
  |---|---|---|---|
  |mysql_1|docker-entrypoint.sh mysqld|Up|0.0.0.0:32769->3306/tcp|

### Exports
Expose ports without publishing them to the host machine - they’ll only be accessible to linked services. Only the internal port can be specified.

Ports are not exposed to host machines, only exposed to other services.

mysql:
  image: mysql:5.7
  expose:
    - "3306"
If I do docker-compose ps, it will look like:

  |Name|Command|State|Ports|
  |---|---|---|---|
  |mysql_1|docker-entrypoint.sh mysqld|Up|3306/tcp|
