# 20250408-阿里云服务器+域名+SSL+RaiDrive+Docker部署Alist+WebDav映射本地文件夹

# 1. Docker前置工作

## ubuntu安装Docker

```bash
# 1. 更新软件包索引并安装必要的依赖
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 2. 添加Docker的官方GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 3. 设置Docker的稳定版仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. 更新apt包索引并安装Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# 5. 将当前用户添加到docker用户组（可选，这样可以不用sudo运行docker命令）
sudo usermod -aG docker $USER

# 6. 启动Docker服务
sudo systemctl start docker

# 7. 设置Docker开机自启
sudo systemctl enable docker

# 8. 验证Docker安装是否成功
docker --version
```

> 新版本的 Docker 默认包含了 docker compose 命令（注意没有连字符）

## 给Docker设置镜像加速器

因为默认的镜像源被屏蔽，如果不设置镜像加速器，会出现镜像拉不下来的情况：`Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)`。

1. 创建或修改 daemon.json 文件

```bash
# 创建目录（如果不存在）
sudo mkdir -p /etc/docker

# 创建或编辑 daemon.json 文件
sudo vim /etc/docker/daemon.json
```

2. 在 daemon.json 中添加以下内容

   ```json
   {
     "registry-mirrors": [
       "https://docker.registry.cyou",
       "https://docker-cf.registry.cyou",
       "https://dockercf.jsdelivr.fyi",
       "https://docker.jsdelivr.fyi",
       "https://dockertest.jsdelivr.fyi",
       "https://mirror.aliyuncs.com",
       "https://dockerproxy.com",
       "https://mirror.baidubce.com",
       "https://docker.m.daocloud.io",
       "https://docker.nju.edu.cn",
       "https://docker.mirrors.sjtug.sjtu.edu.cn",
       "https://docker.mirrors.ustc.edu.cn",
       "https://mirror.iscas.ac.cn",
       "https://docker.rainbond.cc"]
   }
   ```

3. 重启 Docker 服务以使配置生效

```bash
# 重新加载 Docker 守护进程
sudo systemctl daemon-reload

# 重启 Docker 服务
sudo systemctl restart docker
```

4. 验证配置是否生效

   ```bash
   # 查看 Docker 信息，包括镜像源配置
   docker info
   ```

## 常用docker命令

- 如果需要重启docker:

```bash
# 停止服务
sudo systemctl stop docker
sudo systemctl stop containerd
```

- 查看正在运行的容器

  ```bash
  docker ps
  docker ps -a # 查看所有容器（包括已停止的）
  ```

- 查看容器详细信息

  ```bash
  # 查看特定容器的详细信息
  docker inspect 容器ID或名称
  
  # 查看容器资源使用情况
  docker stats
  ```

- 查看容器日志

  ```bash
  # 查看容器日志
  docker logs 容器ID或名称
  
  # 实时查看日志
  docker logs -f 容器ID或名称
  
  # 查看最后 100 行日志
  docker logs --tail 100 容器ID或名称
  ```

  

# 2. 安装 AList

## 创建数据目录

建议创建持久化存储目录以保留配置：

```bash
   mkdir -p /etc/alist
```

## Docker命令行部署

1. 拉取Alist镜像

```bash
docker pull xhofe/alist:latest
```

2. 启动容器

   ```bash
   docker run -d \
   --restart=unless-stopped \
   -v /etc/alist:/opt/alist/data \
   -p 5244:5244 \
   -e PUID=0 \
   -e PGID=0 \
   -e UMASK=022 \
   -e TZ=Asia/Shanghai \
   --name="alist" \
   xhofe/alist:latest
   ```

   - `-v /etc/alist:/opt/alist/data`：将本地目录挂载到容器内，实现配置持久化。
   - `-p 5244:5244`：容器端口映射，左侧可自定义主机端口。
   - `TZ=Asia/Shanghai`：设置容器时区。

## docker compose部署

1. **创建docker-compose.yml文件**

```yml
version: '3.3'
services:
  alist:
    image: 'xhofe/alist:beta'
    container_name: alist
    volumes:
      - '/etc/alist:/opt/alist/data'
    ports:
      - '5244:5244'
    environment:
      - PUID=0
      - PGID=0
      - UMASK=022
    restart: unless-stopped
```

2. **启动服务**

```bash
   docker-compose up -d
```

具体看：[使用 Docker | AList文档](https://alist.nn.ci/zh/guide/install/docker.html#安装)

> 参考文章：[Docker部署 Alist - 一壶缘 - 博客园](https://www.cnblogs.com/yihuyuan/p/18773312)


## 获取管理员密码

- **查看初始密码**

```bash
   docker exec -it alist ./alist admin
```

- 对于v3.25.0及以上版本，密码可能随机生成，需使用：

```bash
docker exec -it alist ./alist admin random  # 生成随机密码
docker exec -it alist ./alist admin set NEW_PASSWORD  # 手动设置密码
```

## 更新与维护

1. **更新Alist版本**

   ```bash
      docker stop alist && docker rm alist  # 停止并删除旧容器
      docker pull xhofe/alist:latest        # 拉取最新镜像
      docker run ...（原启动命令）          # 重新部署
   ```

2. **备份与恢复**
   定期备份 `/etc/alist` 目录，恢复时重新挂载即可。

 

至此，Alist已经在服务器中运行成功，接下来需要配Nginx反向代理、域名和SSL证书

# 申请SSL证书

在阿里云申请个人测试证书，每个证书有效期3个月。你也可以用一些自动申请和续期的脚本。

![image-20250408174022248](https://s2.loli.net/2025/04/08/yZC2eoaWiuPAb7w.png)

可以使用Let's Encrypt获取免费SSL证书：

```bash
sudo apt install certbot python3-certbot-nginx 
sudo certbot --nginx -d pan.example.com
```

把证书放到nginx配置的目录中，并配置好路径。

# Nginx反向代理配置

给本地docker中的alist容器的5244端口设置代理，当访问域名时转发到Alist服务。

1. 创建配置文件

   ```bash
   sudo mkdir -p /etc/nginx/sites-available/
   sudo vim /etc/nginx/sites-available/alist
   ```
   
   ```nginx
   server
    {
        listen 80;
        listen [::]:80;
        server_name your-site; # 你的域名
        return 301 https://$host$request_uri;
   }
   server
    {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name pan.space-x.space;
        charset utf-8;
   
        ssl_certificate /etc/nginx/ssl/xxx.pem;
        ssl_certificate_key /etc/nginx/ssl/xxx.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        ssl_ciphers "TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
        ssl_session_cache builtin:1000 shared:SSL:10m;
        # openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
        #ssl_dhparam /etc/nginx/ssl/dhparam.pem;
   
        location / {
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header Host $host:$server_port;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header Range $http_range;
          proxy_set_header If-Range $http_if_range;
          proxy_redirect off;
          proxy_pass http://127.0.0.1:5244;
          # the max size of file to upload
          client_max_body_size 20000m;
        }
   
        access_log /var/log/nginx/alist-access.log;
        error_log /var/log/nginx/alist-error.log;
    }
```



2. 创建符号链接到 sites-enabled

   ```bash
   sudo ln -s /etc/nginx/sites-available/alist /etc/nginx/sites-enabled/
```

3. 测试语法并重启

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

此时理论上已经可以访问域名就可以进入到alist页面了，但是还差了一步，配置DNS解析，不然浏览器打不开域名的。


# 配置域名DNS解析

在阿里云的域名控制台，点击你的域名管理，进入云解析DNS控制面板。

点击添加记录，选择记录类型为A，主机记录就是二级域名的名字，记录值填服务器的IP地址。

![image-20250408173328997](https://s2.loli.net/2025/04/08/2vyadBFYGQIrxlA.png)

配置好后就能正常访问了。

# 使用RaiDrive+WebDav映射到本地

安装RaiDrive软件，添加NAS中的WebDav，路径填`/dav`，账号密码跟Alist一致。

![image-20250408181957607](https://s2.loli.net/2025/04/08/lDs1EBaZyxdnSi5.png)

![image-20250408181900236](https://s2.loli.net/2025/04/08/9KmXkwdUt2RnW4x.png)