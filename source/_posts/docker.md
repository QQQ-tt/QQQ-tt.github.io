## 1.docker安装

````shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
````

## 2.镜像源配置（可选项）

更改配置后重启docker

### 2.1 docker hub镜像源

````shell
# 查看文件内容
cat /etc/docker/daemon.json
````

````shell
# 添加以下内容（可选项）
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://ccr.ccs.tencentyun.com"
    ]
}
EOF
````

| 镜像来源 |                 地址                 |
| :------: | :----------------------------------: |
|  网易云  |     https://hub-mirror.c.163.com     |
|  百度云  |     https://mirror.baidubce.com      |
|  阿里云  | https://0lah620j.mirror.aliyuncs.com |

### 2.2 私有仓库源

向daemon.json文件添加以下内容

````
{
	  "insecure-registries": ["ip:port"]
}
````

## 3.开放2375 port

````shell
vim /usr/lib/systemd/system/docker.service
````

````shell
# ExecStart 最后添加
-H tcp://0.0.0.0:2375
````

````shell
# 重启
systemctl daemon-reload
systemctl restart docker
````

````shell
# 查看端口情况
netstat -ano | grep 2375
````

## 4.部分docker命令

### 4.1 删除全部none镜像

```shell
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

### 4.2 如果镜像在运行

````shell
# 停止容器
docker stop $(docker ps -a | grep "Exited" | awk '{print $1 }') 
# 删除容器
docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }')
# 删除镜像
docker rmi $(docker images | grep "none" | awk '{print $3}')
````

### 4.3 根据容器名称删除容器及镜像

````shell
#!/bin/bash
container_id=`docker ps -aq --filter "ancestor=$1"`
echo $container_id
docker stop $container_id
docker rm $container_id
image_id=`docker image ls -q -f "reference=$1"`
echo $image_id
docker rmi $image_id
````

### 4.4 导入导出

#### 4.4.1 save

> docker save [options] images [images...]

示例

````shell
docker save -o xxx.tar xxx:latest
docker save > xxx.tar xxx:latest
````

#### 4.4.2 load

> docker load [options]

示例

`````shell
docker load -i xxx.tar
docker load < xxx.tar
`````

#### 4.4.3 export

> docker export [options] container

示例

````shell
docker export -o xxx.tar xxx
````

#### 4.4.4 import

> docker import [options] file|URL|- [REPOSITORY[:TAG]]

````shell
docker import xxx:tar xxx:tag
cat xxx.tar | docker import - xxx:tag
````

#### 4.4.5 区别

- export命令导出的tar文件略小于save命令导出的
- export命令是从容器（container）中导出tar文件，而save命令则是从镜像（images）中导出
- 基于第二点，export导出的文件再import回去时，无法保留镜像所有历史（即每一层layer信息）

### 4.5  查看容器信息

````shell
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
````

````shell
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器
````

其中`{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}`是go语言的模板`{{  }}`是开始和结束，`range`是开始遍历，`end`是结束遍历

### 4.5 进入容器

````shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
````

````shell
docker exec -it 容器 /bin/bash 
````

### 4.6 容器运行挂载目录

```shell
# 最后结尾没有 '/' 可以是目录也可以是文件
/home/qtx/mysql/my.cnf:/etc/my.cnf
# 最后结尾有 '/' 代表是目录
/home/qtx/mysql/:/var/lib/mysql/
```