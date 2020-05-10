# Docker Networks Defaults  
- Each container connected to a private virtual network 'bridge'  
- Each virtual network routes through NAT firewall on host IP  
- All containers on a virtual network can talk to each other without -p  
- Best partice is to create a new virtual network for each app  
    ex1) my_web_app network for mysql + php + apache container  
    ex2) my_api network for mongo + nodejs container 
  
# Customizable  
- Default works well in many cases, but easy to swap  
- New virtual networks  
- Attach containers to more than one virtual networks  
- Skip virtual networks and use host IP (--net=host)  
- Use different Docker network drivers to gain new abilities  
  
# Exmaple  
docker container run -p 80:80 --name webhost -d nginx  
docker container port webhost  
docker container inspect --format '{{ .NetworkSettings.IPAddress}}' webhost  
ifconfig 
  
# CLI management  
show networks - docker network ls  
| NETWORK ID   | NAME   | DRIVER | SCOPE |
|--------------|--------|--------|-------| 
| 5984d6e43b28 | bridge | bridge | local | 
| 95d7e49ec99b | host   | host   | local | 
| b665ccf89f49 | none   | null   | local | 
* bridge: default docker virtual network, which is NAT'ed behind the host IP  
  Containers on the default bridge network can only access each other by IP addresses, unless you use the --link option, which is considered legacy. On a user-defined bridge network, containers can resolve each other by name or alias.
  [(read more)](https://docs.docker.com/network/bridge/)
* host: gains performance by skipping virtual networks but sacrifices security of contianer model  
* none: removes eth0 and only leaves you with localhost interface in container  
  
create a network - docker network create --driver  
ex) docker network create my_app_net  
ex) docker container run -d --name new_nginx --network my_app_net nginx
| NETWORK ID  | NAME       | DRIVER  | SCOPE  |
|-------------|------------|---------|--------|
| 5984d6e43b28| bridge     | bridge  | local  | 
| 95d7e49ec99b| host       | host    | local  |
| b500e485d947| my_app_net | bridge  | local  | 
| b665ccf89f49| none       | null    | local  |
* driver: built-in or 3rd party extensions that give you virtual network features  

inspect a network - docker network inspect  
ex) docker network inspect bridge  
ex) docker network inspect my_app_net  

attach a network to container - docker network connect  
dynamically creates a NIC in a container on an existing virtual network
ex) docker container run -d --name my_container nginx
ex) docker netowrk connect my_app_net my_container
ex) docker network inspect my_app_net
```
"Containers": {
    "0c6ee6e0342f119900f186fd36df2660adcbd09daaafa2be45c68c831ab68060": {
        "Name": "my_container",
        "EndpointID": "78465f022ef4b846cd89ba6e3f0565d74621059a48181659fc529089f16f213e",
        "MacAddress": "02:42:ac:12:00:03",
        "IPv4Address": "172.18.0.3/16",
        "IPv6Address": ""
    },
    "cf2bcbb9c761cd7176c1aa7dfc1e6e671b6e689d97a28760c7bdef4d4e83fff6": {
        "Name": "my_nginx",
        "EndpointID": "b613a93076bc5752cd69bc151e3715f5daeea3e4b53e6f3619749e5dee5b920e",
        "MacAddress": "02:42:ac:12:00:02",
        "IPv4Address": "172.18.0.2/16",
        "IPv6Address": ""
    }
},
```
ex) docker container inspect my_container
```
       "Networks": {
                "bridge": {
                    ......
                },
                "my_app_net": {
                    ......
                }
```
ex) docker container exec -it my_container ping my_nginx
* By using the 'docker exec -it [container] sh' (or bash) command on a container, we can connect to a shell from inside it.

detach a network from container - docker network disconnect  
ex) docker netowrk disconnect my_app_net my_container
  
