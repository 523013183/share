## 环境

```
centos 7.8
```

### 安装 thumbor

```shell
#查看python 版本
python -v
#安装python的包管理工具pip
yum install epel-release
yum install python-devel
yum install python-pip
pip install --upgrade pip #升级pip版本
#安装编译工具
yum install gcc
#安装thumbor
pip install thumbor

#创建运行thumbor的用户：
groupadd thumbor
useradd -g thumbor -s /sbin/nologin thumbor
#创建thumbor的配置文件
mkdir -p /etc/thumbor
thumbor-config > /etc/thumbor/thumbor.conf
#创建service 文件
vi /etc/systemd/system/thumbor.service

​```
[Unit]
Description=thumbor
After=network.target

[Service]
ExecStart=/usr/bin/thumbor \
	    --port=8199 \
	    --conf=/etc/thumbor/thumbor.conf
User=thumbor
Restart=on-failure
Type=simple
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
​```
#启动
systemctl start thumbor
systemctl enable thumbor

```

### 访问地址nginx配置  /etc/nginx/conf.d/thumbor.conf

```shell
#这里访问的是minio的存储服务地址
server {
        listen       8008;
        server_name  192.168.2.1;
        location ~/([A-Za-z0-9._-]+)/(.*)$ {
                proxy_http_version 1.1;
                proxy_set_header Connection "keep-alive";
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass http://127.0.0.1:8199/unsafe/$2/http://127.0.0.1:9000/test/$1;
        }
        location ~/([A-Za-z0-9._-]+)$ {
                proxy_http_version 1.1;
                proxy_set_header Connection "keep-alive";
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass http://127.0.0.1:9000/test/$1;
        }
}
```

