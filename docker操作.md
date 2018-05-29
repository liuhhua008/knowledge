### **基本概念**

Docker 包括三个基本概念

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

先理解了这三个概念，就理解了 Docker 的整个生命周期。

```shell
#先查看先决条件
[root@localhost ~]# ls -l /sys/class/misc/device-mapper/

总用量 0

-r--r--r--. 1 root root 4096 5月  21 12:12 dev

drwxr-xr-x. 2 root root    0 5月  21 12:12 power

lrwxrwxrwx. 1 root root    0 5月  21 10:44 subsystem -> ../../../../class/misc

-rw-r--r--. 1 root root 4096 5月  21 12:12 uevent

[root@localhost ~]# uname -a

Linux localhost.localdomain 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

[root@localhost ~]# 
```



### **1、docker安装与启动**

```shell
yum install -y epel-release
#yum源安装
yum install docker-io # 安装docker
# 配置文件 /etc/sysconfig/docker

#官网安装
[root@localhost ~]# curl -sSL https://get.docker.com/ | sh

chkconfig docker on  # 加入开机启动
root@localhost ~]# systemctl enbale docker


service docker start # 启动docker服务 或者如下：
root@localhost ~]# systemctl start docker

# 基本信息查看
docker version # 查看docker的版本号，包括客户端、服务端、依赖的Go等
docker info # 查看系统(docker)层面信息，包括管理的images, containers数等
docker pull centos 下载
docker images [ centos ] 查看
docker run -i -t centos /bin/bash12345678910111213
```

### **2、镜像的获取与容器的使用**

```
# 搜索镜像
docker search <image> # 在docker index中搜索image
# 下载镜像
docker pull <image>  # 从docker registry server 中下拉image
# 查看镜像 
    docker images： # 列出images
    docker images -a # 列出所有的images（包含历史）
    docker rmi  <image ID>： # 删除一个或多个image12345678
# 使用镜像创建容器
docker run -i -t sauloal/ubuntu14.04
docker run -i -t sauloal/ubuntu14.04 /bin/bash # 创建一个容器，让其中运行 bash 应用，退出后容器关闭
docker run -itd --name centos_aways --restart=always centos #创建一个名称centos_aways的容器，自动重启
# --restart参数：always始终重启；on-failure退出状态非0时重启；默认为，no不重启

# 查看容器
    docker ps ：列出当前所有正在运行的container
    docker ps -l ：列出最近一次启动的container
    docker ps -a ：列出所有的container（包含历史，即运行过的container）
    docker ps -q ：列出最近一次运行的container ID
# 再次启动容器
    docker start/stop/restart <container> #：开启/停止/重启container
    docker start [container_id] #：再次运行某个container （包括历史container）
#进入正在运行的docker容器
    docker exec -it [container_id] /bin/bash
    docker run -i -t -p <host_port:contain_port> #：映射 HOST 端口到容器，方便外部访问容器内服务，host_port 可以省略，省略表示把 container_port 映射到一个动态端口。

# 删除容器
    docker rm <container...> #：删除一个或多个container
    docker rm `docker ps -a -q` #：删除所有的container
    docker ps -a -q | xargs docker rm #：同上, 删除所有的container12345678910111213141516171819202122
```

docker run 和 docker create 参数基本一样，run是创建容器并后台启动，create是只创建容器。 
docker run 相当于docker create 和 docker start

```
run创建容器：docker run -itd
create创建： docker create -it
    -t, --tty                       Allocate a pseudo-TTY
    -i, --interactive               Keep STDIN open even if not attached
    -d, --detach                    Run container in background and print container ID #run的参数12345
```

**容器资源限制参数**

```
-m 1024m --memory-swap=1024m  # 限制内存最大使用（bug：超过后进程被杀死）
--cpuset-cpus="0,1"           # 限制容器使用CPU
```

**docker容器随系统自启参数**

```
docker run --restart=always redis
```

- **no** – 默认值，如果容器挂掉不自动重启
- **on-failure** – 当容器以非 0 码退出时重启容器

- - 同时可接受一个可选的最大重启次数参数 (e.g. **on-failure:5**).
- **always** – 不管退出码是多少都要重启

```
docker run -itd --name test01 -p IP:sport:dport  -m 1024m --memory-swap=1024m --cpuset-cpus="0,1" --restart=always <image ID> 
docker exec -it test01 bash  # 进入容器也可以用exec命令
```

**查看容器状态信息**

```
[root@localhost ~]# docker stats 
[root@localhost ~]# docker stats --no-stream
```

### **进入容器 - nsenter 命令**

**nsenter安装** 
nsenter 工具在 util-linux 包2.23版本后包含。 如果系统中 util-linux 包没有该命令，可以按照下面的方法从源码安装。

```
cd /usr/src ; wget https://www.kernel.org/pub/linux/utils/util-linux/v2.28/util-linux-2.28.tar.gz
./configure --without-ncurses
make nsenter && sudo cp nsenter /usr/local/bin
```

**nsenter使用** 
nsenter 可以访问另一个进程的名字空间。nsenter 要正常工作需要有 root 权限。 
为了连接到容器，你还需要找到容器的第一个进程的 PID，可以通过下面的命令获取。

```
PID=$(docker inspect --format "{{ .State.Pid }}" <container>)1
```

通过这个 PID，就可以连接到这个容器：

```
nsenter --target $PID --mount --uts --ipc --net --pid1
```

更简单的，建议下载 .bashrc_docker，并将内容放到 .bashrc 中。

```
wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc12
```

这个文件中定义了很多方便使用 Docker 的命令，例如 docker-pid 可以获取某个容器的 PID；而 
docker-enter 可以进入容器或直接在容器内执行命令。

```
echo $(docker-pid <container>)
docker-enter <container> ls
docker-enter <container> bash
```

### **3、持久化容器与镜像**

#### **3.1 通过容器生成新的镜像**

运行中的镜像称为容器。你可以修改容器（比如删除一个文件），但这些修改不会影响到镜像。不过，你使用docker commit 命令可以把一个正在运行的容器变成一个新的镜像。

```
docker commit <container> [repo:tag] # 将一个container固化为一个新的image，后面的repo:tag可选。1
```

#### **3.2 持久化容器**

export命令用于持久化容器

```
docker export <CONTAINER ID> > /tmp/export.tar1
```

#### **3.3 持久化镜像**

Save命令用于持久化镜像

```
docker save 镜像ID > /tmp/save.tar1
```

**3.4 导入持久化container**

删除container 2161509ff65e

```
docker rm 2161509ff65e1
```

导入export.tar文件

```
cat /tmp/export.tar | docker import - export:latest
```

#### **3.5 导入持久化image**

删除image daa11948e23d

```
docker rmi daa11948e23d1
```

导入save.tar文件

```
docker load < /tmp/save.tar1
```

对image打tag

```
docker tag daa11948e23d load:tag1
```

#### **3.6 export-import与save-load的区别**

导出后再导入(export-import)的镜像会丢失所有的历史，而保存后再加载（save-load）的镜像没有丢失历史和层(layer)。这意味着使用导出后再导入的方式，你将无法回滚到之前的层(layer)，同时，使用保存后再加载的方式持久化整个镜像，就可以做到层回滚。（可以执行docker tag 来回滚之前的层）。

#### **3.7 一些其它命令**

```
 docker logs $CONTAINER_ID #查看docker实例运行日志，确保正常运行
    docker inspect $CONTAINER_ID #docker inspect <image|container> 查看image或container的底层信息
    docker build <path> 寻找path路径下名为的Dockerfile的配置文件，使用此配置生成新的image
    docker build -t repo[:tag] 同上，可以指定repo和可选的tag
    docker build - < <dockerfile> 使用指定的dockerfile配置文件，docker以stdin方式获取内容，使用此配置生成新的image
    docker port <container> <container port> 查看本地哪个端口映射到container的指定端口，其实用docker ps 也可以看到123456
```

### **一些使用技巧**

#### **docker文件存放目录**

Docker实际上把所有东西都放到/var/lib/docker路径下了。

```
[root@localhost docker]# ls -F
containers/  devicemapper/  execdriver/  graph/  init/  linkgraph.db  repositories-devicemapper  volumes/12
```

containers目录当然就是存放容器（container）了，graph目录存放镜像，文件层（file system layer）存放在graph/imageid/layer路径下，这样我们就可以看看文件层里到底有哪些东西，利用这种层级结构可以清楚的看到文件层是如何一层一层叠加起来的。



### 3. 开启Docker的远程访问功能

+ **开启远程访问**

1. 修改docker.service 文件在[Service]中加下面两行。

   ExecStart=
   ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

```shell

[root@bogon docker.service.d]# vim /usr/lib/systemd/system/docker.service 

[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target rhel-push-plugin.socket registries.service
Wants=docker-storage-setup.service
Requires=docker-cleanup.timer

[Service]
Type=notify
NotifyAccess=all
EnvironmentFile=-/run/containers/registries.conf
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
Environment=DOCKER_HTTP_HOST_COMPAT=1
Environment=PATH=/usr/libexec/docker:/usr/bin:/usr/sbin
ExecStart=/usr/bin/dockerd-current \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
"/usr/lib/systemd/system/docker.service" 41L, 1350C                         1,1          顶端
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target rhel-push-plugin.socket registries.service
Wants=docker-storage-setup.service
Requires=docker-cleanup.timer

[Service]
Type=notify
NotifyAccess=all
EnvironmentFile=-/run/containers/registries.conf
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
Environment=DOCKER_HTTP_HOST_COMPAT=1
Environment=PATH=/usr/libexec/docker:/usr/bin:/usr/sbin
ExecStart=/usr/bin/dockerd-current \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          $REGISTRIES
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=0
Restart=on-abnormal
KillMode=process
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

"/usr/lib/systemd/system/docker.service" 43L, 1440C 已写入                  
[root@bogon docker.service.d]# 

```

  

- **重启Docker，并且打开防火墙端口:** 

  ```shell
  [root@bogon docker.service.d]# systemctl daemon-reload 
  [root@bogon docker.service.d]# systemctl restart docker
  [root@bogon docker.service.d]# ps -ef|grep docker
  root      7471     1  0 21:04 ?        00:00:00 /usr/bin/dockerd-current -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
  root      7476  7471  0 21:04 ?        00:00:00 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
  root      7570  5035  0 21:04 pts/1    00:00:00 grep --color=auto docker
  [root@bogon docker.service.d]# firewall-cmd --zone=public --add-port=2375/tcp --permanent
  success
  [root@bogon docker.service.d]# firewall-cmd --reload
  success
  [root@bogon docker.service.d]# 
  ```

  

- 源码所在机器，即客户机配置一条环境变量DOCKER_HOST=tcp://ip:2375。ip替换成自己的机器ip，这样客户机就能远程访问Docker环境了。

### 4.maven连接Docker服务器

+ **插件介绍**

​      Maven有个插件，叫dockerfile-maven-plugin，这是[它的地址](https://github.com/spotify/dockerfile-maven) ，它会连接远程Docker，只要一个命令就能把本地的jar包打成Docker镜像，命令执行完毕后，服务器上就会有刚打包好的镜像，此时再执行该镜像即可。

​	对了，它有个前生哥哥，叫docker-maven-plugin，同一个作者出品，同一个味道。百度“Spring Docker”除了Spring自家demo,其它大部分都是用这个老插件实现。这个老插件允许没有DockerFile，相关配置参数全部写在pom.xml中。后来作者觉得这样不好，很多人因此都不写DockerFile了，然后又搞的他更新这个插件很累，因为得时时同步DockerFile的新特性（他本人没说过后半句话，我臆测的-_-），所以作者废弃了它，重写了这个dockerfile-maven-plugin插件。

+ **pom文件配置**

 eureka项目下的pom.xml加上dockerfile插件，目前最新版本是1.4,${docker.image.prefix}变量需要在pom的properties内定义   

```xml
<properties>
        <docker.image.prefixdemo</docker.image.prefix>
    </properties>

<build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.0</version>
                <configuration>
                    <repository>${docker.image.prefix}/${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
        </plugins>
</build>

```





- **DockerFile配置** 
  在项目根目录下添加一个Dockerfile 文件，Dockerfile拼写有错误，大小写也不能有错 ！镜像文件该打成什么样子的就是读取这个文件中的配置，位置就在工程的target目录下面.

  Dockerfile文件内容如下：

  ```
  # dockerfile 基础配置
  FROM daocloud.io/library/java:8u40-b22
  VOLUME /tmp
  ARG JAR_FILE
  ADD ${JAR_FILE} /app/app.jar
  WORKDIR /app/
  EXPOSE 8889
  ENTRYPOINT ["java","-jar","./app.jar"]
  ```

  上面各个参数的含义，要自己学习一下。 
  **上面DockerFile中由于用了java:8的layer层，本身它就有600多M，所以打出来的镜像肯定是超600M的，如果想瘦身一点，可以替换成openjdk,参照spring 官网写的spring boot docker demo**

  

- **打包发布为远程docker镜像，执行如下命令：**

```
mvn clean package dockerfile:build -DskipTests
```

+ **在服务器上查看镜像:**

这里可以看到新创建的镜像，以及他的依赖镜像也被传上去了。

```shell
[root@bogon docker.service.d]# docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
liu008/spring-boot-liu     latest              d75b2d815fc3        8 seconds ago       847 MB
docker.io/busybox          latest              8ac48589692a        6 weeks ago         1.15 MB
daocloud.io/library/java   8u40-b22            0a5e1e22983a        3 years ago         815 MB
[root@bogon docker.service.d]# 
```



+ **执行运行镜像命令**

```
sudo docker run --name discovery -p 8800:8800 -d --it 镜像名1 --privileged=true
```

启动参数解释： 
–name: 指定容器名，可省略 
-p: 容器端与宿主端口绑定，不可省略，否则容器外应用没法访问这个应用 
-d: 后台运行

--privileged=true   加权限 不加会报什么iptabls的错，但Centos7上没有iptables

最后执行下面命令查看容器是否启动成功

```
sudo docker ps
```


# docker常用命令

### 1.容器命令

```shell
#查看所有容器，不带a是显示正在运行的，带了就是所有的那怕是运行失败的 
[root@bogon docker.service.d]# docker ps -a
CONTAINER ID        IMAGE                    COMMAND                 CREATED             STATUS              PORTS               NAMES
8c079617e7f5        liu008/spring-boot-liu   "java -jar ./app.jar"   2 minutes ago       Created                                 liu008
#删除容器
[root@bogon docker.service.d]# docker rm 8c079617e7f5 -f
#删除所有容器
[root@bogon docker.service.d]# rm `docker ps -a -q`

#创建容器   app是容器名  8888是外部访问端口 -d是后台挂机运行 liu008/spring-boot-liu是镜像名字
[root@localhost sysconfig]# docker run --name app -d -p 8888:8080 -it liu008/spring-boot-liu --privileged=true

#进入容器
[root@localhost sysconfig]# docker attach app

#查看容器配置
ot@localhost sysconfig]# docker inspect app

#查看容器的日志
[root@localhost sysconfig]# docker logs app

#退出容器
EXIT  直接退出  ctrl + p  再 ctrl + q  就是退出但是 容量继续保持后台远行。
```



###2.一些坑

　容量不运行，报什么$PATH找不到，经过一番排查，如下解决方案有用：

`root@etcd1 k8s]``# cd /usr/libexec/docker/`

`[root@etcd1 docker]``# ln -s docker-runc-current docker-runc`



 容量内要访问外部数据库被拒，这其实不是DOCKER的问题。是mysql本身的安全问题。

1. 在装有MySQL的机器上登录MySQL mysql -u root -p密码
2. 执行`use mysql;`
3. 执行`update user set host = '%' where user = 'root';`这一句执行完可能会报错，不用管它。
4. 执行`FLUSH PRIVILEGES;`