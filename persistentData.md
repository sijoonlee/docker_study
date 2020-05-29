# Container is usually immutable and ephemeral
- What about database? or unique data? -> Docker provides the way: Persistent data

### Persistent data
- Volume : special location outside of container union file system
- Bind Mounts : link container path to host path


# Volume
- VOLUME command in Dockerfile
ex) mysql
https://hub.docker.com/layers/mysql/library/mysql/latest/images/sha256-3857435f3cf9de798f86a88708e27b17b8c6b4b281279e758f399e22f3a81b17?context=explore
    ``` 
    VOLUME [/var/lib/mysql]
    ```
docker pull mysql
docker image inspect mysql
    ```
    "Volumes": {
        "/var/lib/mysql": {}
    },
    ```

docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True mysql
docker container inspect mysql
    ```
    "Mounts": [
        {
            "Type": "volume",
            "Name": "1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654",
            "Source": "/var/lib/docker/volumes/1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654/_data",
            "Destination": "/var/lib/mysql",
            "Driver": "local",
            "Mode": "",
            "RW": true,
            "Propagation": ""
        }
    ],
    ```
docker container logs mysql

docker volume ls
DRIVER              VOLUME NAME
local               1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654
docker volume inspect 1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654       
[
    {
        "CreatedAt": "2020-05-29T06:44:50-07:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654/_data",
        "Name": "1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654",
        "Options": null,
        "Scope": "local"
    }
]

### Named volume
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql
docker volume ls
DRIVER              VOLUME NAME
local               1a9e3309dc49cfd0d14910135d22c034de952a405bc7defcf0e0faebdaaec654
local               mysql-db

docker volume inspect mysql-db
[
    {
        "CreatedAt": "2020-05-29T06:51:10-07:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql-db/_data",
        "Name": "mysql-db",
        "Options": null,
        "Scope": "local"
    }
]


### docker volume create
- to use custom drivers and labels
- need to run this before 'docker run'
    ```
    Usage:  docker volume create [OPTIONS] [VOLUME]
    Options:
    -d, --driver string   Specify volume driver name (default "local")
        --label list      Set metadata for a volume
    -o, --opt map         Set driver specific options (default map[])
    ```

# Bind Mounting
- Maps a host file or directory to a container file or directory
- two locations pointing to the same file(s)
- skips UFS, host files overwritten over container files
- can't use in Dockerfile
- use at container run
    ```
    ... run -v /users/abc/efg:/path/conatiner (max/linus)
    ... run -v //c/users/abc/efg:/path/container (windows)
    ```
- example
```
    # local pwd contains index.html
    # host's index.html will be used over container's index.html
    # therefore, nginx will use local index.html over its own defaul index.html
    docker container run -d --name nginx -p 80:80 -v %{pwd}:/user/share/nginx/html nginx
```
