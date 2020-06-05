# 1 Nodes Demo

### docker info
- check if docker swarm is enabled
```
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.9
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc version: dc9208a3303feef5b3839f4323d9beb36df0a9dd
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.3.0-55-generic
 Operating System: Ubuntu 19.10
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 7.748GiB
 Name: ubuntu
 ID: QRNG:N67P:YH5A:VYMZ:GVJR:TD6K:VRLS:OOPW:7BRH:SOPB:A2QF:FO3U
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: No swap limit support
```

### docker swarm init
- lots of PKI and security automation
    - Root signing certificate created for Swarm
    - Certificate is issued for first manager node
    - join tokens are created
- Raft database created to store root CA, configs and secrets
    - encrypted by default on disk (1.13+)
    - no need for another key/value system to hold orchestration
    - replicates logs amongst managers via mutual TLS in 'control plane'
```
Swarm initialized: current node (k9iimzoijgtruinet5wv8d7ri) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0j3q6gca3ev3qoza4q20666hilqmdaprpl43rlcdztetp7i4re-b6mjwja3bgw8e4vwjbqwjrl6s 192.168.64.128:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### docker node ls
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
k9iimzoijgtruinet5wv8d7ri *   ubuntu              Ready               Active              Leader              19.03.9
```

### docker node --help
```
Usage:	docker node COMMAND

Manage Swarm nodes

Commands:
  demote      Demote one or more nodes from manager in the swarm
  inspect     Display detailed information on one or more nodes
  ls          List nodes in the swarm
  promote     Promote one or more nodes to manager in the swarm
  ps          List tasks running on one or more nodes, defaults to current node
  rm          Remove one or more nodes from the swarm
  update      Update a node

Run 'docker node COMMAND --help' for more information on a command.
```

### docker service --help
```
Usage:	docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.
```

### docker service create alpine ping 8.8.8.8
```
8xcym9ew34zs0qmuxqoiqhnq4
overall progress: 1 out of 1 tasks 
1/1: running   
verify: Service converged 
```
- note that "8xcym9ew34zs0qmuxqoiqhnq4" is not container id
- it is service id  

### docker service ls
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
8xcym9ew34zs        relaxed_rubin       replicated          1/1                 alpine:latest  
```

### docker service ps relaxed_rubin
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
nznw0lnrdjqr        relaxed_rubin.1     alpine:latest       ubuntu              Running             Running 8 minutes ago  
```
- note that the name of the process is different from the name of the service

### docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d1f601352919        alpine:latest       "ping 8.8.8.8"      12 minutes ago      Up 12 minutes                           relaxed_rubin.1.nznw0lnrdjqrdlvwhgp4ct46n

### docker service update relaxed_rubin --replicas 3
```
relaxed_rubin
overall progress: 3 out of 3 tasks 
1/3: running   
2/3: running   
3/3: running   
verify: Service converged 
```

### docker service ps relaxed_rubin
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
nznw0lnrdjqr        relaxed_rubin.1     alpine:latest       ubuntu              Running             Running 27 minutes ago                           
oedpfws28tb2        relaxed_rubin.2     alpine:latest       ubuntu              Running             Running about a minute ago                       
l9ms8ey4no39        relaxed_rubin.3     alpine:latest       ubuntu              Running             Running about a minute ago                       
sijoonlee@ubuntu:~$ 

```

### docker container rm -f relaxed_rubin.1.nznw0lnrdjqrdlvwhgp4ct46n
--> will delete one of 3
--> Swarm will make new one to achieve 3/3
--> docker service ps relaxed_rubin
```
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR                         PORTS
zbyw70mn2j5x        relaxed_rubin.1       alpine:latest       ubuntu              Running             Running about a minute ago                                 
nznw0lnrdjqr         \_ relaxed_rubin.1   alpine:latest       ubuntu              Shutdown            Failed about a minute ago    "task: non-zero exit (137)"   
oedpfws28tb2        relaxed_rubin.2       alpine:latest       ubuntu              Running             Running 5 minutes ago                                      
l9ms8ey4no39        relaxed_rubin.3       alpine:latest       ubuntu              Running             Running 5 minutes ago   
```

### docker service rm relaxed_rubin
- remove service

### docker service update --help
```
Usage:	docker service update [OPTIONS] SERVICE

Update a service

Options:
      --args command                       Service command args
      --config-add config                  Add or update a config file on a service
      --config-rm list                     Remove a configuration file
      --constraint-add list                Add or update a placement constraint
      --constraint-rm list                 Remove a constraint
      --container-label-add list           Add or update a container label
      --container-label-rm list            Remove a container label by its key
      --credential-spec credential-spec    Credential spec for managed service account (Windows only)
  -d, --detach                             Exit immediately instead of waiting for the service to
                                           converge
      --dns-add list                       Add or update a custom DNS server
      --dns-option-add list                Add or update a DNS option
      --dns-option-rm list                 Remove a DNS option
      --dns-rm list                        Remove a custom DNS server
      --dns-search-add list                Add or update a custom DNS search domain
      --dns-search-rm list                 Remove a DNS search domain
      --endpoint-mode string               Endpoint mode (vip or dnsrr)
      --entrypoint command                 Overwrite the default ENTRYPOINT of the image
      --env-add list                       Add or update an environment variable
      --env-rm list                        Remove an environment variable
      --force                              Force update even if no changes require it
      --generic-resource-add list          Add a Generic resource
      --generic-resource-rm list           Remove a Generic resource
      --group-add list                     Add an additional supplementary user group to the container
      --group-rm list                      Remove a previously added supplementary user group
                                           from the container
      --health-cmd string                  Command to run to check health
      --health-interval duration           Time between running the check (ms|s|m|h)
      --health-retries int                 Consecutive failures needed to report unhealthy
      --health-start-period duration       Start period for the container to initialize before
                                           counting retries towards unstable (ms|s|m|h)
      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)
      --host-add list                      Add a custom host-to-IP mapping (host:ip)
      --host-rm list                       Remove a custom host-to-IP mapping (host:ip)
      --hostname string                    Container hostname
      --image string                       Service image tag
      --init                               Use an init inside each service container to forward
                                           signals and reap processes
      --isolation string                   Service container isolation mode
      --label-add list                     Add or update a service label
      --label-rm list                      Remove a label by its key
      --limit-cpu decimal                  Limit CPUs
      --limit-memory bytes                 Limit Memory
      --log-driver string                  Logging driver for service
      --log-opt list                       Logging driver options
      --mount-add mount                    Add or update a mount on a service
      --mount-rm list                      Remove a mount by its target path
      --network-add network                Add a network
      --network-rm list                    Remove a network
      --no-healthcheck                     Disable any container-specified HEALTHCHECK
      --no-resolve-image                   Do not query the registry to resolve image digest and
                                           supported platforms
      --placement-pref-add pref            Add a placement preference
      --placement-pref-rm pref             Remove a placement preference
      --publish-add port                   Add or update a published port
      --publish-rm port                    Remove a published port by its target port
  -q, --quiet                              Suppress progress output
      --read-only                          Mount the container's root filesystem as read only
      --replicas uint                      Number of tasks
      --replicas-max-per-node uint         Maximum number of tasks per node (default 0 = unlimited)
      --reserve-cpu decimal                Reserve CPUs
      --reserve-memory bytes               Reserve Memory
      --restart-condition string           Restart when condition is met ("none"|"on-failure"|"any")
      --restart-delay duration             Delay between restart attempts (ns|us|ms|s|m|h)
      --restart-max-attempts uint          Maximum number of restarts before giving up
      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)
      --rollback                           Rollback to previous specification
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h)
      --rollback-failure-action string     Action on rollback failure ("pause"|"continue")
      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback
      --rollback-monitor duration          Duration after each task rollback to monitor for
                                           failure (ns|us|ms|s|m|h)
      --rollback-order string              Rollback order ("start-first"|"stop-first")
      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0
                                           to roll back all at once)
      --secret-add secret                  Add or update a secret on a service
      --secret-rm list                     Remove a secret
      --stop-grace-period duration         Time to wait before force killing a container
                                           (ns|us|ms|s|m|h)
      --stop-signal string                 Signal to stop the container
      --sysctl-add list                    Add or update a Sysctl option
      --sysctl-rm list                     Remove a Sysctl option
  -t, --tty                                Allocate a pseudo-TTY
      --update-delay duration              Delay between updates (ns|us|ms|s|m|h)
      --update-failure-action string       Action on update failure ("pause"|"continue"|"rollback")
      --update-max-failure-ratio float     Failure rate to tolerate during an update
      --update-monitor duration            Duration after each task update to monitor for failure
                                           (ns|us|ms|s|m|h)
      --update-order string                Update order ("start-first"|"stop-first")
      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to
                                           update all at once)
  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])
      --with-registry-auth                 Send registry authentication details to swarm agents
  -w, --workdir string                     Working directory inside the container
```

# 3 Nodes Demo

### https://labs.play-with-docker.com/
Create 3 instances (node1, node2, node3)

### docker swarm init --advertise-addr 192.168.0.8
- ifconfig
```
eth0      Link encap:Ethernet  HWaddr 66:4E:0A:3A:48:1D  
          inet addr:192.168.0.8  Bcast:0.0.0.0  Mask:255.255.254.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1296 (1.2 KiB)  TX bytes:0 (0.0 B)
```
- result
```
Swarm initialized: current node (pvpjfletkl5tdhqqk0hjvg34h) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4jcph1wdydt5mmoi5x7zmq3qyah2j5o9z6poyjfbthxc9b71sd-cb7efxbx2xaihpzb0s14jzsbo 192.168.0.8:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
### docker node ls
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
pvpjfletkl5tdhqqk0hjvg34h *   ubuntu              Ready               Active              Leader              19.03.9
```
### Go to Node2

### docker swarm join --token SWMTKN-1-4jcph1wdydt5mmoi5x7zmq3qyah2j5o9z6poyjfbthxc9b71sd-cb7efxbx2xaihpzb0s14jzsbo 192.168.0.8:2377
```
This node joined a swarm as a worker.
```

### Go to Node1

### docker node ls
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
y0z8zeos0g2g1qpop6qhl1kq5 *   node1               Ready               Active              Leader              19.03.4
g2a1j38nyesmbrgvlxiw7ebac     node2               Ready               Active                                  19.03.4
```

### docker node update --role manager node2
- 'docker node ls' to check, note the change at "MANAGER STATUS"
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
y0z8zeos0g2g1qpop6qhl1kq5 *   node1               Ready               Active              Leader              19.03.4
g2a1j38nyesmbrgvlxiw7ebac     node2               Ready               Active              Reachable           19.03.4
```
### docker node --help
```
Usage:	docker node COMMAND

Manage Swarm nodes

Commands:
  demote      Demote one or more nodes from manager in the swarm
  inspect     Display detailed information on one or more nodes
  ls          List nodes in the swarm
  promote     Promote one or more nodes to manager in the swarm
  ps          List tasks running on one or more nodes, defaults to current node
  rm          Remove one or more nodes from the swarm
  update      Update a node

Run 'docker node COMMAND --help' for more information on a command.
```

### docker swarm join-token manager
```
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4jcph1wdydt5mmoi5x7zmq3qyah2j5o9z6poyjfbthxc9b71sd-8l319znuj7wugbvk1wjgxs8sa 192.168.0.8:2377
```

### Go to node3

### docker swarm join --token SWMTKN-1-4jcph1wdydt5mmoi5x7zmq3qyah2j5o9z6poyjfbthxc9b71sd-8l319znuj7wugbvk1wjgxs8sa 192.168.0.8:2377
```
This node joined a swarm as a manager
```
- 'docker node ls' to check
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
y0z8zeos0g2g1qpop6qhl1kq5     node1               Ready               Active              Leader              19.03.4
g2a1j38nyesmbrgvlxiw7ebac     node2               Ready               Active              Reachable           19.03.4
11tjfnz4knmagjz9guks390gg *   node3               Ready               Active              Reachable           19.03.4
```

### Go to node1

### docker service create alpine ping 8.8.8.8
```
vobxqoufaxobhjab0zkza7sf0
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

### $ docker node ps
```
ID                  NAME                       IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
3w7jbv99ggc7        compassionate_bhaskara.1   alpine:latest       node1               Running             Running about a minute ago 
```

### docker service ls
```
ID                  NAME                     MODE                REPLICAS            IMAGE               PORTS
vobxqoufaxob        compassionate_bhaskara   replicated          1/1                 alpine:latest 
```

### docker service ps compassionate_bhaskara
```
ID                  NAME                       IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
3w7jbv99ggc7        compassionate_bhaskara.1   alpine:latest       node1               Running             Running 2 minutes ago 
```

### Go to node3

### docker node ps
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE       ERROR               PORTS
```
- note that node3 has nothing

### docker service create --replicas 3 alpine ping 8.8.8.8
```
koktcwitw429o0z87oping6mq
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```
- docker service ls to check
```
ID                  NAME                     MODE                REPLICAS            IMAGE               PORTS
vobxqoufaxob        compassionate_bhaskara   replicated          1/1                 alpine:latest       
koktcwitw429        wonderful_albattani      replicated          3/3                 alpine:latest 
```
- docker service ps koktcwitw429
```
ID                  NAME                    IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
43wgy9a3hfy6        wonderful_albattani.1   alpine:latest       node3               Running             Running about a minute ago                       
pofvth9vbv0s        wonderful_albattani.2   alpine:latest       node2               Running             Running about a minute ago                       
jydny18v8spk        wonderful_albattani.3   alpine:latest       node1               Running             Running about a minute ago
```

# Scaling out

*Networking overview from official documentation*
- https://docs.docker.com/network/
```
Network drivers
Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality:

bridge: The default network driver. If you don’t specify a driver, this is the type of network you are creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate. See bridge networks.

host: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly. host is only available for swarm services on Docker 17.06 and higher. See use the host network.

overlay: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers. See overlay networks.

macvlan: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. Using the macvlan driver is sometimes the best choice when dealing with legacy applications that expect to be directly connected to the physical network, rather than routed through the Docker host’s network stack. See Macvlan networks.

none: For this container, disable all networking. Usually used in conjunction with a custom network driver. none is not available for swarm services. See disable container networking.

Network plugins: You can install and use third-party network plugins with Docker. These plugins are available from Docker Hub or from third-party vendors. See the vendor’s documentation for installing and using a given network plugin.
```
- https://docs.docker.com/network/overlay/
```
Use overlay networks

The overlay network driver creates a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it (including swarm service containers) to communicate securely when encryption is enabled. Docker transparently handles routing of each packet to and from the correct Docker daemon host and the correct destination container.

When you initialize a swarm or join a Docker host to an existing swarm, two new networks are created on that Docker host:

an overlay network called ingress, which handles control and data traffic related to swarm services. When you create a swarm service and do not connect it to a user-defined overlay network, it connects to the ingress network by default.
a bridge network called docker_gwbridge, which connects the individual Docker daemon to the other daemons participating in the swarm.
You can create user-defined overlay networks using docker network create, in the same way that you can create user-defined bridge networks. Services or containers can be connected to more than one network at a time. Services or containers can only communicate across networks they are each connected to.

Although you can connect both swarm services and standalone containers to an overlay network, the default behaviors and configuration concerns are different. For that reason, the rest of this topic is divided into operations that apply to all overlay networks, those that apply to swarm service networks, and those that apply to overlay networks used by standalone containers.

By default, swarm services which publish ports do so using the routing mesh. When you connect to a published port on any swarm node (whether it is running a given service or not), you are redirected to a worker which is running that service, transparently. Effectively, Docker acts as a load balancer for your swarm services. Services using the routing mesh are running in virtual IP (VIP) mode. Even a service running on each node (by means of the --mode global flag) uses the routing mesh. When using the routing mesh, there is no guarantee about which Docker node services client requests.
```



## Overlay Multi-host Networking
- use --driver overlay when creating network
- container-to-container traffic inside a single Swarm
- optional IPSec(AES) encryption on network creation
- each service can be connected to multiple networks

### Start new session @https://labs.play-with-docker.com/ 
- add 3 new instnaces
- swarm init like before
- make 2 other managers
- Go to Node1

### docker network create --driver overlay mydrupal
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
2d3ea92c3867        bridge              bridge              local
d52a1bc7f3f2        docker_gwbridge     bridge              local
4df4e2fb5df6        host                host                local
4q2b51uk9kje        ingress             overlay             swarm
w4dzwj75fjqz        mydrupal            overlay             swarm
c1742ea77d41        none                null                local
```

### docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=mypass postgres
```
z4npo1trbp2fm6jkebcfseou5
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
```
$ docker service ps psql
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
v2n8yasw8i0t        psql.1              postgres:latest     node1               Running             Running 25 seconds ago  
```
```
$ docker logs psql.1.v2n8yasw8i0ttuekh7orjvuvo 
The files belonging to this database system will be owned by user "postgres".
...
...
2020-06-02 15:51:45.449 UTC [1] LOG:  database system is ready to accept connections
```

### docker service create --name drupal --network mydrupal -p 80:80 drupal
```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
rkncrv3m61vu        drupal              replicated          1/1                 drupal:latest       *:80->80/tcp
z4npo1trbp2f        psql                replicated          1/1                 postgres:latest 
```
```
$ docker service ps drupal
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
rpwjt2pwgpll        drupal.1            drupal:latest       node2               Running             Running about a minute ago   
```
- note that psql is running on node1
- note that drupal is running on node2
- go to Drupal server page
- set the database name as 'psql'
- even though drupal is working on node1, when you access ip address of node2 or node3, the drupal webserver works

## Routing Mesh
- Route ingress (incoming) packets for a service to proper task
- spans all nodes in Swarm
- use IPVS from linux kernel
- load balances Swarm services across their tasks
- two ways this works
    - container-to-container in an Overlay network (uses Virtual IP)
    - external traffic incoming to published ports (all nodes listen)
- stateless load balancing, which is OSI layer 3(TCP), but not layer 4(DNS)

# How swarm works
https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/
