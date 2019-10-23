
>Docker学习笔记 对官方文档的学习 https://docs.docker.com/get-started/


### 安装

Docker 从 17.03 版本之后分为2个版本
* CE (Community Edition: 社区版)
* EE (Enterprise Edition: 企业版)

### 原理

* C/S架构
  * Client - 控制台
  * Server - daemon


|对比项|Docker|VM|
|:--:|----|----|
|实现|Docker Engine|必须使用Hypervisor实现硬件资源的虚拟化|
|操作系统|宿主机OS(共享)|在宿主机OS之上运行虚拟机Guest OS|
|存储空间|小|大(.vmdk .vmi)|
|部署速度|秒级|分钟级|
|移植性|灵活轻便|与虚拟化技术的耦合度高|

### 配置

Docker国内镜像源
```
https://docker.mirrors.ustc.edu.cn
https://hub-mirror.c.163.com
```

linux - 服务管理
```
service docker start       # 启动 docker 服务，守护进程
service docker stop        # 停止 docker 服务
service docker status      # 查看 docker 服务状态
chkconfig docker on        # 设置为开机启动
```


### 基本概念


* Docker 镜像(Docker images)
  * 作用与特点
    * 镜像是只读模板. 一个image可以生成多个容器(实例).
    * 组成 - 每一个镜像由一系列的层(layers)组成.
* Docker 仓库(Docker registeries)
  * 作用与特点
    * 存放镜像.
  * 分类
    * public repository - 公开仓库
    * private repository - 私有仓库
* Docker 容器(Docker containers)
  * 作用与特点
    * 容器都是由镜像创建,容器是镜像的运行实例.
    * 一个容器包含了某个应用运行所需要的环境.
    * 独立性 - 不同容器之间相互隔离.
  * 操作容器
    * 查看容器 `docker ps`
    * 容器状态 `docker stats NAMEorID`
    * 启动容器 `docker start NAMEorID`
    * 重启容器 `docker restart NAMEorID`
    * 停止容器 `docker stop NAMEorID`
    * 强停容器 `docker kill NAMEorID`
    * 删除容器 `docker rm NAMEorID`

### 命令

```
Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/Users/lin/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/Users/lin/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/Users/lin/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/Users/lin/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
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
```

举例如下
```
# 版本信息
docker --version
docker version
# 详细信息
docker info
```

#### docker run

使用镜像来创建一个新的容器 并运行该容器
```
docker run --help

Usage:	docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network string                 Connect a container to a network (default "default")
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```


例:使用nginx镜像创建容器 并运行该容器
```
# 命令格式
# docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 使用nginx镜像 启动一个新的容器(如果镜像不存在 则自动pull该镜像)
docker run --name my-nginx -d -p 8000:80 nginx

# 解释OPTIONS
# --name my-nginx 给该容器命名.  不使用该选项 则给该容器一个随机名
# -d 在"后台"运行容器 并打印出容器ID
# -p 8000:80 将容器中的端口暴露到物理机 访问物理机的8000端口 即访问容器的80端口


# 查看容器日志 my-nginx是容器名称
docker logs my-nginx
```


例:使用nginx镜像创建容器 并运行该容器 (交互式shell)
```
docker run -it 镜像ID
docker run -it 镜像名

# 情况1
docker run -it nginx /bin/bash
root@899b26e72d55:/#
# 按ctrl+p+q 退出容器的shell  而不会停止容器(仍在后台运行)  之后可重新进入容器的shell
docker attach 899b26
# 或
docker exec -it 899b26

# 情况2
docker run -it nginx /bin/bash
root@899b26e72d55:/#
# 终端中输入exit 则停止该容器
```

#### docker images

查看当前已有的镜像
```
docker images --help

Usage:	docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```

例如
```
# 搜索镜像 (OFFICIAL代表该镜像由官方发布)
docker search nginx

# 拉取镜像 使用标签说明版本号 默认拉取最新版本 即:latest
docker pull nginx
docker pull nginx:latest

# 查看当前已有的镜像
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              e445ab08b2be        3 days ago          126MB
httpd               latest              ee39f68eb241        2 weeks ago         154MB


# 查看当前已有的镜像 - 只显示容器ID(便于与其他命令结合使用 写shell脚本等)
docker images -q

# 查看当前已有的镜像 - 显示 摘要信息 和 完整的IMAGE ID
docker images -a --digests --no-trunc
```


#### docker container

容器管理
```
docker container --help

Usage:	docker container COMMAND

Manage containers

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  inspect     Display detailed information on one or more containers
  kill        Kill one or more running containers
  logs        Fetch the logs of a container
  ls          List containers
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  prune       Remove all stopped containers
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  run         Run a command in a new container
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker container COMMAND --help' for more information on a command.
```



重启容器

`docker restart --help` 等价于 `docker container restart --help`

```
docker restart --help


Usage:	docker container restart [OPTIONS] CONTAINER [CONTAINER...]

Restart one or more containers

Options:
  -t, --time int   Seconds to wait for stop before killing the container (default 10)
```


#### docker ps

查看当前已有的容器

`docker ps [OPTIONS]` 等价于 `docker container ls [OPTIONS]`

```
docker container ls --help

Usage:	docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
```


例如
```
# 列出容器 - 正在运行的容器
docker ps
# 二者等价 可互相替换
docker container ls

# 列出容器 - 历史上最近一次创建过的容器(包括所有状态)
docker ps -l

# 列出容器 - 所有容器(正在运行+历史上曾经运行过的)
docker ps -a

# 列出容器 - 所有容器(正在运行+历史上运行过的)    quiet mode 只显示容器ID
docker ps -aq

# 列出容器 - 最近3个创建过的容器
docker ps -n 3
```

#### docker stats

查看容器状态
```
# 查看某个容器的状态
docker stats my-nginx

# 结果实时刷新
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
6f80e11ffb06        my-nginx            0.00%               1.816MiB / 1.952GiB   0.09%               3.32kB / 2.72kB     0B / 0B             2
```

#### docker exec

在容器中执行命令
```
Usage:	docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
```

例:建立与某容器的交互shell
```
# 在交互式shell输入exit退出交互式shell  容器仍然在后台运行
docker exec -it my-nginx /bin/bash

# -w 指定在某个目录
docker exec -w "/etc" -it my-nginx /bin/bash

# -e 设置环境变量
```

例:在容器899b中 执行一条命令 并返回结果
```
# 不进入容器 直接在物理机的命令行执行
docker exec -t 899b ls /etc
```

#### docker attach

```
docker attach --help

Usage:	docker attach [OPTIONS] CONTAINER

Attach local standard input, output, and error streams to a running container

Options:
      --detach-keys string   Override the key sequence for detaching a container
      --no-stdin             Do not attach STDIN
      --sig-proxy            Proxy all received signals to the process (default true)
```

#### docker cp

从容器中复制文件 到物理机
```
docker cp --help

Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

Copy files/folders between a container and the local filesystem

Options:
  -a, --archive       Archive mode (copy all uid/gid information)
  -L, --follow-link   Always follow symbol link in SRC_PATH
```

例如
```
# 容器 -> 物理机
# 把容器899b26中的文件/tmp/1.txt  复制到 物理机~/Downloads/Mac.txt
docker cp 899b26:/tmp/1.txt ~/Downloads/Mac.txt

# 物理机 -> 容器
# 把物理机的~/Downloads/Mac.txt  复制到 容器899b26中的/tmp/new.txt
docker cp ~/Downloads/Mac.txt 899b26:/tmp/new.txt
```


#### docker port

查看容器的端口映射情况
```
Usage:	docker port CONTAINER [PRIVATE_PORT[/PROTO]]

List port mappings or a specific mapping for the container


# 查看某个容器内"所有端口"的映射情况
docker port my-nginx
80/tcp -> 0.0.0.0:8000

# 查看某个容器内"某个端口"的映射情况
docker port my-nginx 80
```

#### docker inspect

查看容器内的细节
```
docker inspect --help

Usage:	docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

返回json字符串 内容包括:
```
"Id":
"Created":
"Path":
"Args":
"State":
"Image":
"ResolvConfPath":
"HostnamePath":
"HostsPath":
"LogPath":
"Name":
"RestartCount":
"Driver":
"Platform":
"MountLabel":
"ProcessLabel":
"AppArmorProfile":
"ExecIDs":
"HostConfig":
"GraphDriver":
"Mounts":
"Config":
"NetworkSettings":
```



#### docker top

查看容器内的进程信息
```
docker top --help

Usage:	docker top CONTAINER [ps OPTIONS]

Display the running processes of a container
```

例如
```
docker top f53079bb4f54

PID                 USER                TIME                COMMAND
5225                root                0:00                /bin/bash
```

#### docker kill

强制结束容器
```
docker kill --help

Usage:	docker kill [OPTIONS] CONTAINER [CONTAINER...]

Kill one or more running containers

Options:
  -s, --signal string   Signal to send to the container (default "KILL")
```


例如
```
docker kill containerNAMEorID
```

经常会发现container中无法结束PID为1的进程,因为结束它等于结束容器

#### docker logs

查看容器内的日志
```
docker logs --help

Usage:	docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
      --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
```

例如
```
# 查看容器日志 - 实时持续查看日志
docker logs my-nginx -f

# 查看容器日志 -t 在每条日志数据之前显示"容器中"系统的时间戳 如2019-07-26T08:02:04.351634542Z
docker logs my-nginx -t

# 筛选 - 绝对时间 - 只查看某个具体时间点之后的日志
docker logs my-nginx -t --since "2019-07-26T08:02:09.369242551Z"

# 筛选 - 绝对时间 - 只查看某个具体时间点之前的日志
docker logs my-nginx -t --until 2019-07-26T08:02:04.369242551Z


# 筛选 - 相对时间 - 只查看"日志最早的时间点"到"50分钟之前的时间点"的日志
docker logs my-nginx --until 50m

# 筛选 - 相对时间 - 只查看"70分钟之前的时间点"到"现在时间点"的日志
docker logs my-nginx -t --since 70m


# 查看日志文件 最后X行的内容(通常是最早的内容)
docker logs my-nginx --tail 5


# 更多选项查看docker logs --help
```

#### docker history

查看已下载的某个镜像的历史
```
Usage:	docker history [OPTIONS] IMAGE

Show the history of an image

Options:
      --format string   Pretty-print images using a Go template
  -H, --human           Print sizes and dates in human readable format (default true)
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```

例如
```
docker history httpd
```


#### docker rm

删除容器
```
Usage:	docker rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove the volumes associated with the container
```

例如
```
# 删除容器 - 一个或多个
docker rm CONTAINER1 CONTAINER1

# 删除容器 - 删除全部容器
docker rm $(docker ps -aq)
# 或
docker ps -aq | xargs docker rm 

# 删除容器 - 强制删除正在运行的容器(使用SIGKILL)
docker rm -f my-nginx

# 删除指定的链接
docker rm --link my-nginx

# 删除卷 - 删除与容器关联的卷
docker rm -v my-nginx
```


#### docker rmi

删除镜像
```
Usage:	docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

例如
```
# 删除镜像
docker rmi 镜像名1 镜像名2

# 删除镜像 - 强制删除
docker rmi -f 镜像ID

# 删除镜像 - 强制删除所有容器!!
docker rmi -f $(docker images -qa)
```

#### docker diff

查看容器中"文件"和"目录"的变化
```
Usage:  docker diff CONTAINER

Inspect changes to files or directories on a container's filesystem
```

例如
```
docker diff my-nginx

C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
C /run
A /run/nginx.pid
```


### Dockerfile

https://docs.docker.com/get-started/part2/


images被Dockerfile定义


### 常用镜像

```
# apache
docker pull httpd

```
