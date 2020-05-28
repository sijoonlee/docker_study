# DEMO 1  
* --rm Automatically remove the container when it exits   

docker container run --rm it centos:7 bash    
yum update curl  
  
docker container run --rm it ubuntu:14.04 bash  
apt-get update && apt-get install -y curl  

  
# DEMO 2 - DNS round robin test
* Multiple containers on a created network can respond to the same DNS address

* Round-robin DNS from [wiki](https://en.wikipedia.org/wiki/Round-robin_DNS  )
```
Round-robin DNS  is a technique of load distribution, load balancing, or fault-tolerance provisioning multiple, redundant Internet Protocol service hosts, e.g., Web server, FTP servers, by managing the Domain Name System's (DNS) responses to address requests from client computers according to an appropriate statistical model.[1]

In its simplest implementation, round-robin DNS works by responding to DNS requests not only with a single potential IP address, but with a list of potential IP addresses corresponding to several servers that host identical services.[2][3] The order in which IP addresses from the list are returned is the basis for the term round robin. With each DNS response, the IP address sequence in the list is permuted.[4] Usually, IP clients initially attempt connections with the first address returned from a DNS query,[5] so that on different connection attempts, clients would receive service from different providers, thus distributing the overall load among servers.
```

* about --net-alias from [stackoverflow](https://stackoverflow.com/questions/36048897/difference-between-link-and-alias-in-overlay-docker-network)
```
There are two differences between --net-alias and --link:

With --net-alias, one container can access the other container only if they are on the same network. In other words, in addition to --net-alias foo and --net-alias bar, you need to start both containers with --net foobar_net after creating the network with docker network create foobar_net.
With --net-alias foo, all containers in the same network can reach the container by using its alias foo. With --link, only the linked container can reach the container by using the name foo.
Historically, --link was created before libnetwork and all network-related features. Before libnetwork, all containers ran in the same network bridge, and --link only added names to /etc/hosts. Then, custom networks were added and the behavior of --link in user-defined networks was changed.

See also Legacy container links for more information on --link.
```


**Create a new virtual network (default bridge driver)**  
docker network create dude

**Create two containers from elasticsearch:2 image**  
docker container run -d --net dude --net-alias search elasticsearch:2
docker container run -d --net dude --net-alias search elasticsearch:2

**nslookup from Alpine**
docker container run --rm --net dude alpine nslookup search

**curl from CentOs**
docker container run --rm --net dude centos curl -s search:9200

--> the response would be signaled from one of the elasticsearch containers

