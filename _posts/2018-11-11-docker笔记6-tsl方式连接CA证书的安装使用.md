---
layout: post
title: docker笔记6-docker笔记6-tsl方式连接CA证书的安装使用
tags:
- docker 


categories: docker
description:  当我们将docker的端口暴露在外网会非常不安全，时刻面临着挖矿，被攻击的威胁。
---
记录使用tsl连接docker， CA证书的安装使用
<!-- more -->

#### 生成安裝证书
参照Docker 服务 TLS 证书全自动生成 https://segmentfault.com/a/1190000012510820

Bash 脚本代码
```
#!/bin/bash
# 
# Created by L.STONE <web.developer.network@gmail.com>
# -------------------------------------------------------------
# 自动创建 Docker TLS 证书
# -------------------------------------------------------------

# 以下是配置信息
# --[BEGIN]------------------------------

CODE="后缀标识码"
IP="IP 地址"
PASSWORD="密码"
COUNTRY="CN"
STATE="省"
CITY="市"
ORGANIZATION="公司名称"
ORGANIZATIONAL_UNIT="Dev"
COMMON_NAME="$IP"
EMAIL="电子邮件地址"

# --[END]--

# Generate CA key
openssl genrsa -aes256 -passout "pass:$PASSWORD" -out "ca-key-$CODE.pem" 4096
# Generate CA
openssl req -new -x509 -days 365 -key "ca-key-$CODE.pem" -sha256 -out "ca-$CODE.pem" -passin "pass:$PASSWORD" -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME/emailAddress=$EMAIL"
# Generate Server key
openssl genrsa -out "server-key-$CODE.pem" 4096

# Generate Server Certs.
openssl req -subj "/CN=$COMMON_NAME" -sha256 -new -key "server-key-$CODE.pem" -out server.csr

echo "subjectAltName = IP:$IP,IP:127.0.0.1" >> extfile.cnf
echo "extendedKeyUsage = serverAuth" >> extfile.cnf

openssl x509 -req -days 365 -sha256 -in server.csr -passin "pass:$PASSWORD" -CA "ca-$CODE.pem" -CAkey "ca-key-$CODE.pem" -CAcreateserial -out "server-cert-$CODE.pem" -extfile extfile.cnf


# Generate Client Certs.
rm -f extfile.cnf

openssl genrsa -out "key-$CODE.pem" 4096
openssl req -subj '/CN=client' -new -key "key-$CODE.pem" -out client.csr
echo extendedKeyUsage = clientAuth >> extfile.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -passin "pass:$PASSWORD" -CA "ca-$CODE.pem" -CAkey "ca-key-$CODE.pem" -CAcreateserial -out "cert-$CODE.pem" -extfile extfile.cnf

rm -vf client.csr server.csr

chmod -v 0400 "ca-key-$CODE.pem" "key-$CODE.pem" "server-key-$CODE.pem"
chmod -v 0444 "ca-$CODE.pem" "server-cert-$CODE.pem" "cert-$CODE.pem"

# 打包客户端证书
mkdir -p "tls-client-certs-$CODE"
cp -f "ca-$CODE.pem" "cert-$CODE.pem" "key-$CODE.pem" "tls-client-certs-$CODE/"
cd "tls-client-certs-$CODE"
tar zcf "tls-client-certs-$CODE.tar.gz" *
mv "tls-client-certs-$CODE.tar.gz" ../
cd ..
rm -rf "tls-client-certs-$CODE"

# 拷贝服务端证书
mkdir -p /etc/docker/certs.d
cp "ca-$CODE.pem" "server-cert-$CODE.pem" "server-key-$CODE.pem" /etc/docker/certs.d/

# /etc/docker/daemon.json
# {
#   "tlsverify": true,
#   "tlscacert": "/etc/docker/certs.d/ca.pem",
#   "tlscert": "/etc/docker/certs.d/server-cert.pem",
#   "tlskey": "/etc/docker/certs.d/server-key.pem",
#   "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
# }

echo " - 修改 /etc/docker/daemon.json 文件"
cat <<EOF
vi /etc/docker/daemon.json
{
  "tlsverify": true,
  "tlscacert": "/etc/docker/certs.d/ca-$CODE.pem",
  "tlscert": "/etc/docker/certs.d/server-cert-$CODE.pem",
  "tlskey": "/etc/docker/certs.d/server-key-$CODE.pem",
  "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
}
EOF

# 拷贝客户端证书文件
# cp -v {ca,cert,key}.pem ~/.docker

# 客户端远程连接
# docker -H 192.168.1.130:2376 --tlsverify --tlscacert ~/.docker/ca.pem --tlscert ~/.docker/cert.pem --tlskey ~/.docker/key.pem ps -a
echo "docker -H $IP:2376 --tlsverify --tlscacert ~/.docker/ca-$CODE.pem --tlscert ~/.docker/cert-$CODE.pem --tlskey ~/.docker/key-$CODE.pem ps -a"

# 客户端使用 cURL 连接
# curl --cacert ~/.docker/ca.pem --cert ~/.docker/cert.pem --key ~/.docker/key.pem https://192.168.1.130:2376/containers/json
echo "curl --cacert ~/.docker/ca-$CODE.pem --cert ~/.docker/cert-$CODE.pem --key ~/.docker/key-$CODE.pem https://$IP:2376/containers/json"

echo -e "\e[1;32mAll be done.\e[0m"

```

#### 修改docker.service 

vim /usr/lib/systemd/system/docker.service 
```
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376
```

#### 执行
```
systemctl daemon-reload
systemctl restart docker.service

```
#### 将3个pem证书放置到客户端(eg:C:\Users\docker-ca)

下载ca.pem ,cert.pem ,key.pem复制到客户端C:\Users\docker-ca（路径自定义）
#### 复制客户端的证书位置（eg:C:\Users\docker-ca）
如果idea 安装了docker插件配置如图
<img src="{{ site.assets }}/images/2018-11-11/201811111222.png"/>

如果使用的docker-maven-plugin配置证书位置
```
<plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>1.2.0</version>
        <configuration>
            <serverId>docker-hub</serverId>
            ...略
            <dockerCertPath>C:\Users\docker-ca</dockerCertPath>
            ...略
        </configuration>
</plugin>

```
<dockerCertPath>C:\Users\docker-ca</dockerCertPath>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">天空蓝蓝的</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>
