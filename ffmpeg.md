## 环境

```
centos 7.8
```



## 安装

```shell
#安装ffmpeg
yum install java-1.8.0-openjdk* -y #安装java jdk包
yum install epel-release
rpm -v --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum install ffmpeg ffmpeg-devel
#查看是否安装成功
ffmpeg -version
```

