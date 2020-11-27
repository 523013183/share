

## 环境

```
centos 7.8
```

### 安装 minio

```shell
mkdir -p /usr/local/minio/{data,bin,etc}

cd /usr/local/minio/bin && wget https://dl.min.io/server/minio/release/linux-ppc64le/minio
chmod +x minio
#开启9000端口

```

## 单机部署

```shell
#minio配置文件:
#vi /usr/local/minio/etc/minio.conf
MINIO_VOLUMES="/usr/local/minio/data"
MINIO_OPTS="--address :9000"
MINIO_ACCESS_KEY="PcII664xXF3XB0S0"
MINIO_SECRET_KEY="bvq3d0CZtPKJXZfjgjnMUMRk5P"
#开启systemctl服务
#vi /etc/systemd/system/minio.service
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/minio/bin/minio
[Service]
# User and group
User=root
Group=root
EnvironmentFile=/usr/local/minio/etc/minio.conf
ExecStart=/usr/local/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
# Let systemd restart this service always
Restart=always
# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536
# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no
[Install]
WantedBy=multi-user.target

#启动服务
systemctl daemon-reload && systemctl enable minio.service #重载&&开机启动
systemctl start minio && systemctl status minio
```

## 集群部署

```shell
#新建start.sh 
## 单机转集群， 需要把/usr/local/minio/data 原来的磁盘目录都删除了
export MINIO_ACCESS_KEY=PcII664xXF3XB0S0
export MINIO_SECRET_KEY=bvq3d0CZtPKJXZfjgjnMUMRk5P
/usr/local/minio/bin/minio server http://192.168.2.1/usr/local/minio/data http://192.168.2.1/usr/local/minio/data2 \
               http://192.168.2.2/usr/local/minio/data http://192.168.2.2/usr/local/minio/data2 >> /var/log/minio/minio.log 2>&1 &
               
## 启动(多台服务器都要执行)
sh start.sh

## 集群 nginx部署(192.168.2.3)
vi /etc/nginx/conf.d/minio.conf

upstream minio-server
{
        server 192.168.2.1:9000 #Minio服务器1
        weight=25
        max_fails=2
        fail_timeout=30s;
        server 192.168.2.2:9000 #Minio服务器2
        weight=25
        max_fails=2
        fail_timeout=30s;
}
server
{
        listen 8000;
        server_name localhost;
        charset utf-8;
        default_type text/html;
        location /{
                proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-For $remote_addr;
                client_body_buffer_size 10M;
                client_max_body_size 10G;
                proxy_buffers 1024 4k;
                proxy_read_timeout 300;
                proxy_next_upstream error timeout http_404;
                proxy_pass http://minio-server;
        }
}
               
```

## 如需做图片处理，请查看  [thumbor.md](thumbor.md) 

## 如需做音视频处理，请查看  [ffmpeg.md](ffmpeg.md) 



## 如果使用PHP上传则需要设置php、nginx 配置

```
php.ini, upload_max_filesize 和 post_max_size 修改大小
/etc/nginx/nginx.conf   
client_max_body_size 20m; //20M
```
