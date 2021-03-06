# Course Github Repo
https://github.com/BretFisher/udemy-docker-mastery

# check version
docker version

# check information
docker info

# help  
docker --help  
Usage:	docker [OPTIONS] COMMAND  
  
A self-sufficient runtime for containers  
  
Options:  
      --config string      Location of client config files (default "/ home/sijoonlee/.docker")  
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var  
                           and default context set with "docker context use")  
  -D, --debug              Enable debug mode  
  -H, --host list          Daemon socket(s) to connect to  
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")  
      --tls                Use TLS; implied by --tlsverify  
      --tlscacert string   Trust certs signed only by this CA (default "/home/sijoonlee/.docker/ca.pem")  
      --tlscert string     Path to TLS certificate file (default "/home/sijoonlee/.docker/cert.pem")  
      --tlskey string      Path to TLS key file (default "/home/sijoonlee/.docker/key.pem")  
      --tlsverify          Use TLS and verify the remote  
  -v, --version            Print version information and quit  
  
Management Commands:  
  builder     Manage builds  
  config      Manage Docker configs  
  container   Manage containers  
  context     Manage contexts  
  engine      Manage the docker engine  
  image       Manage images  
  network     Manage networks  
  node        Manage Swarm nodes  
  plugin      Manage plugins  
  secret      Manage Docker secrets  
  service     Manage services  
  stack       Manage Docker stacks  
  swarm       Manage Swarm  
  system      Manage Docker  
  trust       Manage trust on Docker images  
  volume      Manage volumes  
  
Commands:  
  attach      Attach local standard input, output, and error streams to a running container  
  build       Build an image from a Dockerfile  
  commit      Create a new image from a container's changes  
  cp          Copy files/folders between a container and the local filesystem  
  create      Create a new container  
  deploy      Deploy a new stack or update an existing stack  
  diff        Inspect changes to files or directories on a container's filesystem  
  events      Get real time events from the server  
  exec        Run a command in a running container  
  export      Export a container's filesystem as a tar archive  
  history     Show the history of an image  
  images      List images  
  import      Import the contents from a tarball to create a filesystem image  
  info        Display system-wide information  
  inspect     Return low-level information on Docker objects  
  kill        Kill one or more running containers  
  load        Load an image from a tar archive or STDIN  
  login       Log in to a Docker registry  
  logout      Log out from a Docker registry  
  logs        Fetch the logs of a container  
  pause       Pause all processes within one or more containers  
  port        List port mappings or a specific mapping for the container  
  ps          List containers  
  pull        Pull an image or a repository from a registry  
  push        Push an image or a repository to a registry  
  rename      Rename a container  
  restart     Restart one or more containers  
  rm          Remove one or more containers  
  rmi         Remove one or more images  
  run         Run a command in a new container  
  save        Save one or more images to a tar archive (streamed to STDOUT by default)  
  search      Search the Docker Hub for images  
  start       Start one or more stopped containers  
  stats       Display a live stream of container(s) resource usage statistics  
  stop        Stop one or more running containers  
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE  
  top         Display the running processes of a container  
  unpause     Unpause all processes within one or more containers  
  update      Update configuration of one or more containers  
  version     Show the Docker version information  
  wait        Block until one or more containers stop, then print their exit codes  
  
Run 'docker COMMAND --help' for more information on a command.  
  
  
# commmand line structure  
- old: docker <command> <options>  
    ex) docker run  
- new: docker <command> <sub-command> <options>  
    ex) docker container run  
      
# Starting with a Nginx web server  
docker container run --publish 8080:80 nginx  
- downlaod image 'nginx' from docker hub  
- start a new container from the image  
- open port 8080 on the host IP  
- routes that traffic to the container IP, port 80  
  
## --detach : run it in the background  
docker container run --publish 8080:80 --detach nginx  
  
## --name : run it with a manually-given name  
docker container run --publish 8080:80 --detach --name webhost nginx  
 
## logs  
docker contaienr logs webhost  
- show logs for a specific container  
  
## top  
docker container top webhost  
- show process in a specific container  
  
## inspect  
docker container inspect  
- details of one container config  
  
## stats  
docker container stats  
- performance stats for all containers  
  
## stop  
docker stop webhost  
  
## rm  
docker container rm 3bf 897 9ca 7f7 9a8 bf8 4e3 06d a3b  
- delete single(multiple) container  
- rm -f : force to remove even when the container is running  
  
## list containers  
docker container ls   
docker ps  
  
## list images  
docker image ls  
  
## list process  
docker run --name mongo -d mongo  
docker ps  
docker top mongo ; list running processes in specific container  
ps aux ; list all processes in system  
ps aux | grep mongo ; filter mongo  
  
## manage mulitple containers  
- nginx  
docker container run -d --name proxy -p 80:80 nginx   
  
- mysql  
docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql  
docker container logs db  
  
- httpd  
docker container run -d --name webserver -p 8080:80 httpd  
  
- clean up  
docker container stop <id> <id> <id>  
docker container rm <id> <id> <id>  
  
- confirm  
docker container ls  
docker image ls  
  
## port  
check port  
docker container port <container>  
  
## -it  
-i: keep session open to receive terminal input  
-t: pseudo-tty, simulates a real terminal like what SSH does  
-a: attach STDOUT/STDERR and forward signals  
  
docker container run -it  : start new container interactively  
docker container exec -it : run additional command in existing container  
  
ex) docker container run -it --name proxy nginx bash  
  
ex) docker container run -it --name ubuntu ubuntu (by default it uses bash)  
ex) docker container run -ai ubuntu  
  
ex) docker container run -it alpine bash ; won't work since bash is not part of alpine  
ex) docker container run -it alpine sh ; works  
   
