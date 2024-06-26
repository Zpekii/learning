# 环境

- OS: Windows

# 安装docker

需要科学上网

- **访问**`docker官方地址`:[Get Started | Docker](https://www.docker.com/get-started/)

- **点击  `Download for Windows` 下载**

# 拉取镜像

需要科学上网

## 最新的nginx镜像

```bat
docker pull nginx:latest
```

## 最新的Ubuntu镜像

```bat
docker pull ubuntu:latest
```



# 准备工作

## 配置Nginx

### 编写配置文件

- 准备创建nginx容器时挂载的卷 `C:XXX\XXXX\nginx\conf.d\` (位置可任意)，在此目录下创建文件 `default.conf` 

- 打开 `default.conf`,根据nginx配置文件语法，编写配置内容

  ```nginx
  server {
    server_name localhost; # 服务主机
  
          listen 6223;  # 监听端口
  
    charset utf-8;
  
    error_page  404              /404.html;
  
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
      root   /var/data/web/50x;
    }
  	
    # 反向代理,将指定路径转发到指定目标, '/api'为 'http' 请求
    location /api {
      resolver 127.0.0.11;
      set $target http://myUbuntu:6167;
      proxy_pass $target;
  
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-for $remote_addr;
  
      port_in_redirect off;
  
      proxy_connect_timeout 300s;
      proxy_send_timeout 300s;
      proxy_read_timeout 300s;
    }
  	
    # '/msg' 为 'websocket' 连接
    location /msg {
      resolver 127.0.0.11;
      set $target http://myUbuntu:6167;
      proxy_pass $target;
      
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-for $remote_addr;
  
      port_in_redirect off;
  
      proxy_connect_timeout 300s;
      proxy_send_timeout 300s;
      proxy_read_timeout 300s;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
  
    }
  
    root /var/tank/fe;
    index  index.html; # 引入前端构建项目,即前端访问页面
  
    location / {
      # First attempt to serve request as file, then
      # as directory, then fall back to redirecting to index.html
      try_files $uri $uri/ $uri.html /index.html;
    }
  
    location ~* \.(?:css|js|jpg|svg)$ {
      expires 30d;
      add_header Cache-Control "public";
    }
  
    location ~* \.(?:json)$ {
      expires 1d;
      add_header Cache-Control "public";
    }
  }
  
  ```

- **请确保配置文件语法正确**

# 创建容器

## 创建Nginx容器

### 停止同名容器，如果有

```bat
docker stop myNginx
```

### 删除同名容器，如果有

```bat
docker rm myNginx
```

### 创建容器

```bat
docker run --name myNginx ^
  -d ^
  --restart=always ^
  -v  "C:XXX\XXXX\nginx\conf.d:/etc/nginx/conf.d" ^
  -v "C:XXX\XXXX\fe:/var/tank/fe" ^
  -p 6223:6223 ^
 --network qnear ^ 
 nginx:latest
```

**如果 `qnear` 未创建,则执行创建:**

```
docker network create qnear
```

- 参数解释:
  - `--name` : 容器名(唯一)
  - `-d`: 在后台运行容器并打印容器ID
  - `--restart=always`:始终在docker客户端运行时重启容器
  -  `-v`:挂载卷，格式为`主机目录:容器目录`，用于将宿主机的目录或文件挂载到容器中，实现共享文件
    - 第一个挂载的卷为`nginx`的配置文件
    - 第二个挂载的卷为前端构建项目 `/fe`下必须有 `index.html`
  - `-p`:端口映射，格式为`主机端口:容器端口`，用于将容器的端口映射到宿主机的端口,可以实现在宿主机访问指定端口来访问容器
  - `--network`:指定容器使用的网络模式, 可以实现多个容器组网，互相访问

## 创建Ubuntu容器(后端运行容器)

### 停止同名容器，如果有

```bat
docker stop myUbuntu
```

### 删除同名容器，如果有

```bat
docker rm myUbuntu
```

### 创建容器

```bat
docker run -it --name myUbuntu ^
	-d ^
	--restart=always ^
	-v "C:XXX\XXXX\be:/var/deploy/tank/be/" ^
	-p 6116:6116 ^
	--network qnear ^
	ubuntu:latest
```

- 参数解释:
  - `-it`:以交互模式运行容器，并为容器分配一个伪终端
  - `-v`:挂载卷，挂载后端项目
  - `-d` `--name` `-p` `--network`: 解释同上

# 其他

## **在进行 `go build` 无法正常下载依赖时**

**报错信息如:**: ` Get "https://proxy.golang.org/github.com/cloudflare/ahocorasick/@v/v0.0.0-20210425175752-730270c3e184.zip": tls: failed to verify certificate: x509: certificate signed by unknown authority`

**解决：**

- 若科学上网无法解决，可尝试再执行以下操作

- 输入: `apt-get install --reinstall ca-certificates`

  或

- 依次输入:

  - `apt install -y ca-certificates`
  - `cd /usr/local/share/ca-certificates`
  - `update-ca-certificates`

