# Docker命令



## docker pull

```
docker pull [镜像]
```

```
docker images // 查看镜像
docker rmi // 删除镜像
docker rm // 删除容器 -f 强制删除
```



## docker ps

```
docker ps // 查看正在运行容器
docker ps -a // 查看所有容器
docker ps -a --format "{{name}}"  		// 只显示name
docker ps -a --format "table {{.ID}}\t{{.Names}}"  		// 显示ID和name
```



## docker  start

```
docker start [容器名或ID]
docker start [container1] [container2] [container3]
```



## docker run

```
docker run [镜像]
docker run -d [镜像] // detached mode
```

```
docker run -p // 端口映射
```

`[exp] docker run -p 80:80 nginx // 宿主机80端口转到container80端口，:前是宿主机`

```
docker run -v // 文件夹映射
```

绑定挂载：`-v`宿主机目录：容器内目录

命名卷挂载：`-v`卷的名字：容器内目录

通过`docker volume create [nginx]`

```
docker run -it --rm alpine // alpine是轻量级liunx系统
-it 进入container
--rm 当container停止时删除container
// 用于临时调试容器
```

```
docker run -d --restart always nginx
--restart always // container停止立即重启
always 任何停止都会重启
unless-stopped 手动停止不会重启
```



## docker rename

```
docker rename [原容器名或ID] [新容器名]
```

```
docker run --name 
```



## docker stop

```
docker stop [容器名或ID]
docker stop [container1] [container2] [container2]
// 强制停止
docker kill [容器名或ID]
```



## docker volume

```
docker volume create [卷名字]
```

```
docker volume ls // 查看所有docker卷
```

```
docker volume inspect [卷名] // 查看卷信息
docker inspect [container] // 查看容器信息
```

```
docker volume rm [卷名]
```

```
docker volume prune -a  // 删除【所有】没有任何container使用的volume
```



> create 完后记得挂载一下
>
> doker run -v nginx_html:/usr/share/nginx/html nginx
>
> 1. nginx_html为create的卷名，后面nginx为镜像

`[exp]`inspect信息如下：

```cmd
D:\>docker volume inspect nginx_html
[
    {
        "CreatedAt": "2025-10-24T03:09:03Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/nginx_html/_data",
        "Name": "nginx_html",
        "Options": null,
        "Scope": "local"
    }
]
```

上面查到的路径（如`/var/lib/docker/volumes/`）是Linux虚拟机内部的路径，而不是Windows或macOS主机的路径。

windows系统下，数据存储在 Docker 虚拟机内，无法像普通 Windows 文件夹一样直接浏览，以下是几种实用的访问方式：

- **Linux**

	> 可以sudo cd [source路径或者inspect完后的Mountpoint路径] 修改查看

- ***通过容器内部查看（Windows）***

	```
	docker exec -it [容器名] /bin/bash
	cd <容器内的挂载点路径>
	```

	- 一个重要概念

		```
		(以卷名nginx_html为例)
		宿主机 (Host Machine)
		├── /var/lib/docker/volumes/nginx_html/_data  ← 卷的"源"路径 (Source)
		    └── index.html
		    └── style.css
		
		容器 (Container)
		├── /usr/share/nginx/html  ← 卷的"目标"路径 (Destination)
		    └── index.html  (实际是宿主机文件的映射)
		    └── style.css  (实际是宿主机文件的映射)
		```

		**简单来说**：

		- `/var/lib/docker/volumes/nginx_html/_data` → **宿主机上的真实数据位置**
		- `/usr/share/nginx/html` → **容器内访问数据的窗口**

		**如何查看挂载点路径？**(前提是先挂载，不然为Mounts为空)

		- **windows**
		
			```cmd
			docker inspect [container] // 或者管道筛选
			docker inspect [container] | findstr /C:"Source" /C:"Destination"
			```
		
			>  "Source": "/var/lib/docker/volumes/nginx_html/_data",
			>  "Destination": "/usr/share/nginx",
		
		- **Linux**
		
			```
			docker inspect [container] | grep "Mounts"
			```
		
			
		
		## docker exec
		
		```
		docker exec -it [容器名或者ID] Linux命令
		
		/bin/bash // 进入容器内交互
		```
		
		容器为极简操作系统，许多工具需要自己安装(如vim等)
		
		1. **查看container内Linux发行版**
		
			```
			cat /etc/os-release
			```
		
			`[exp]`如下：
		
			```
			PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
			NAME="Debian GNU/Linux"
			VERSION_ID="13"
			VERSION="13 (trixie)"
			VERSION_CODENAME=trixie
			DEBIAN_VERSION_FULL=13.1
			ID=debian
			HOME_URL="https://www.debian.org/"
			SUPPORT_URL="https://www.debian.org/support"
			BUG_REPORT_URL="https://bugs.debian.org/"
			```
		
			发现为Debian，确定安装命令为apt
		
		2. **安装过程**
		
			```
			apt update
			apt install vim //Linux基本命令
			```
		

更多查看: [Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html)



# Docker Dockerfile

> Dockerfile 是一个用来构建镜像的文本文件

| **Dockerfile 指令** |                **说明**                |
| :-----------------: | :------------------------------------: |
|        FROM         |    指定基础镜像，用于后续的指令构建    |
|         RUN         |      在构建过程中在镜像中执行命令      |
|        COPY         |        将文件或目录复制到镜像中        |
|         CMD         | 指定容器创建时的默认命令（可以被覆盖） |
|     ENTRYPOINT      | 设置容器创建时的主要命令（不可被覆盖） |
|       WORKDIR       |         设置后续指令的工作目录         |
|         ENV         |              设置环境变量              |
|       EXPOSE        |            仅仅只是声明端口            |
|       VOLUME        |             定义匿名数据卷             |

指令详解: [DockerFile指令详解](https://www.runoob.com/docker/docker-dockerfile.html)



# Docker Compose

> 运行多容器 Docker 应用程序的工具。通过 Compose，可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务

> 可简单了解：[YAML 入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)

**使用步骤**：

1. 使用 Dockerfile 定义应用程序的环境
2. 使用 docker-compose.yml 定义构成应用程序的服务
3. 执行 docker-compose up 命令来启动并运行整个应用程序

