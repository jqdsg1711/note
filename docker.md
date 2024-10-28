# docker安装

### 卸载旧版

```
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

### 配置Docker的yum库

安装yum工具

```
yum install -y yum-utils
```

配置docker的yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装docker

```
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 启动和校验

```
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```

### 配置阿里源

```Bash
# 创建目录
mkdir -p /etc/docker

# 复制内容，注意把其中的镜像加速地址改成你自己的
tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://s8g542qi.mirror.aliyuncs.com"]
}
EOF

# 重新加载配置
systemctl daemon-reload

# 重启Docker
systemctl restart docker
```



如果阿里源不行可以换一个

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://do.nark.eu.org",
        "https://dc.j8.work",
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```





# 镜像制作

### 1.1将微服务打包

打包成jar包



### 1.2制作镜像

#### 1.2.1编写Dockerfile文件

![image-20231215193453447](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20231215193453447.png)

```
# 基础镜像
FROM openjdk:8-jdk-alpine
# 设定时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 拷贝jar包
COPY ./target/client80-1.0-SNAPSHOT.jar ./app.jar
# 入口
ENTRYPOINT ["java", "-jar","./app.jar"]

```



### 1.3将镜像push到docker hub上

#### 1.3.1登录docker hub

```
docker login
```

#### 1.3.2使用docker tag命令为本地镜像添加新的标签

```
docker tag 2.0 mouranyixiao/client80:1.0   
```

#### 1.3.3使用docker push将新添加的镜像上传到docker hub

```
docker push mouranyixiao/client80:1.0  
```

注意

mouranyixiao是账号，client80是该账号下的镜像库。



# DockerCompose

基本语法

| **docker run 参数** | **docker compose 指令** | **说明**   |
| :------------------ | :---------------------- | :--------- |
| --name              | container_name          | 容器名称   |
| -p                  | ports                   | 端口映射   |
| -e                  | environment             | 环境变量   |
| -v                  | volumes                 | 数据卷配置 |
| --network           | networks                | 网络       |

黑马商城部署文件：

```YAML
version: "3.8"

services:
  mysql:
    image: mysql
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "./mysql/conf:/etc/mysql/conf.d"
      - "./mysql/data:/var/lib/mysql"
      - "./mysql/init:/docker-entrypoint-initdb.d"
    networks:
      - hm-net
  hmall:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: hmall
    ports:
      - "8080:8080"
    networks:
      - hm-net
    depends_on:
      - mysql
  nginx:
    image: nginx
    container_name: nginx
    ports:
      - "18080:18080"
      - "18081:18081"
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/nginx.conf"
      - "./nginx/html:/usr/share/nginx/html"
    depends_on:
      - hmall
    networks:
      - hm-net
networks:
  hm-net:
    name: hmall
```

## 基础命令

基础语法：

```Bash
docker compose [OPTIONS] [COMMAND]
```

其中，OPTIONS和COMMAND都是可选参数，比较常见的有：

![image-20231219192203447](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20231219192203447.png)

```Bash
# 1.进入root目录
cd /root

# 2.删除旧容器
docker rm -f $(docker ps -qa)

# 3.删除hmall镜像
docker rmi hmall

# 4.清空MySQL数据
rm -rf mysql/data

# 5.启动所有, -d 参数是后台启动
docker compose up -d
# 结果：
[+] Building 15.5s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                    0.0s
 => => transferring dockerfile: 358B                                                    0.0s
 => [internal] load .dockerignore                                                       0.0s
 => => transferring context: 2B                                                         0.0s
 => [internal] load metadata for docker.io/library/openjdk:11.0-jre-buster             15.4s
 => [1/3] FROM docker.io/library/openjdk:11.0-jre-buster@sha256:3546a17e6fb4ff4fa681c3  0.0s
 => [internal] load build context                                                       0.0s
 => => transferring context: 98B                                                        0.0s
 => CACHED [2/3] RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo   0.0s
 => CACHED [3/3] COPY hm-service.jar /app.jar                                           0.0s
 => exporting to image                                                                  0.0s
 => => exporting layers                                                                 0.0s
 => => writing image sha256:32eebee16acde22550232f2eb80c69d2ce813ed099640e4cfed2193f71  0.0s
 => => naming to docker.io/library/root-hmall                                           0.0s
[+] Running 4/4
 ✔ Network hmall    Created                                                             0.2s
 ✔ Container mysql  Started                                                             0.5s
 ✔ Container hmall  Started                                                             0.9s
 ✔ Container nginx  Started                                                             1.5s

# 6.查看镜像
docker compose images
# 结果
CONTAINER           REPOSITORY          TAG                 IMAGE ID            SIZE
hmall               root-hmall          latest              32eebee16acd        362MB
mysql               mysql               latest              3218b38490ce        516MB
nginx               nginx               latest              605c77e624dd        141MB

# 7.查看容器
docker compose ps
# 结果
NAME                IMAGE               COMMAND                  SERVICE             CREATED             STATUS              PORTS
hmall               root-hmall          "java -jar /app.jar"     hmall               54 seconds ago      Up 52 seconds       0.0.0.0:8080->8080/tcp, :::8080->8080/tcp
mysql               mysql               "docker-entrypoint.s…"   mysql               54 seconds ago      Up 53 seconds       0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp
nginx               nginx               "/docker-entrypoint.…"   nginx               54 seconds ago      Up 52 seconds       80/tcp, 0.0.0.0:18080-18081->18080-18081/tcp, :::18080-18081->18080-18081/tcp
```



# docker stack



1. **docker stack deploy**: 部署一个新的 Docker 栈或更新现有的 Docker 栈。
   ```
   docker stack deploy -c <compose-file> <stack-name>
   ```

2. **docker stack ls**: 列出当前活动的 Docker 栈。
   ```
   docker stack ls
   ```

3. **docker stack ps**: 列出 Docker 栈中的服务任务。
   ```
   docker stack ps <stack-name>
   ```

4. **docker stack rm**: 删除一个 Docker 栈及其服务。
   ```
   docker stack rm <stack-name>
   ```

5. **docker stack services**: 列出 Docker 栈中的服务。
   
   ```
   docker stack services <stack-name>
   ```
   
6. **docker stack inspect**: 检查 Docker 栈的详细信息。
   
   ```
   docker stack inspect <stack-name>
   ```

​     7.修改服务实例数量

```
docker service scale myapp_servicename=10
```

这些是一些常用的 `docker stack` 命令，它们允许你管理 Docker 栈中的服务及其部署。

# mysql部署

```
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v ./mysql/data:/var/lib/mysql \
  -v ./mysql/conf:/etc/mysql/conf.d \
  -v ./mysql/init:/docker-entrypoint-initdb.d \
  wjqmysql
```



# nacos部署

https://www.jb51.net/article/248585.htm

```
# spring
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
server.tomcat.basedir=
nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://<ip>:<port>/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```



## 开放远程调用接口

```
# 开启端口命令  （--permanent永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-port=2375/tcp --permanent
# 重新载入
firewall-cmd --reload

# 使用 vim 编辑docker服务配置文件
vim /lib/systemd/system/docker.service
# 找到如下配置行
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 将其注释掉或者直接删除，替换成下面的配置行.然后保存退出
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

# 重新加载docker配置并重启服务
systemctl daemon-reload && systemctl restart docker

# 然后直接在命令行客户端输入如下命令，IP地址改为自己的
curl http://192.168.88.129:2375/version
# 或者在浏览器直接访问，IP地址改为自己的
http://192.168.88.129:2375/version

```

[详细](https://blog.csdn.net/qq_53058639/article/details/136163775?ops_request_misc=&request_id=&biz_id=102&utm_term=%E5%90%AF%E8%BF%9C%E7%A8%8Bdocker%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%96%B9%E6%B3%95&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-136163775.142^v100^pc_search_result_base8&spm=1018.2226.3001.4187)

