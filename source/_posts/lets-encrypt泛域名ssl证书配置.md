---
title: " Let's Encrypt泛域名SSL证书配置\t\t"
url: 408.html
id: 408
comments: false
categories:
  - 杂
date: 2019-04-09 09:53:35
tags:
---
参考

[Let's Encrypt泛域名SSL证书申请](https://www.laozuo.org/11668.html)

[快速签发 Let's Encrypt 证书指南](https://www.cnblogs.com/esofar/p/9291685.html)

[acme.sh 配合 letsencrypt 配置泛域名](https://www.jianshu.com/p/dbe180979e77)

[acme.sh的GitHub](https://github.com/Neilpang/acme.sh)
## 安装

必备软件

```bash
yum install -y curl socat cron
```

安装acme.sh

```bash
curl https://get.acme.sh | sh
```

整个安装过程进行了以下几步，了解一下即可：

1. 把 acme.sh 安装到当前用户的主目录`$HOME`下的`.acme.sh`文件夹中，即`~/.acme.sh/`，之后所有生成的证书也会放在这个目录下；
2. 创建了一个指令别名`alias acme.sh=~/.acme.sh/acme.sh`，这样我们可以通过`acme.sh`命令方便快速地使用 acme.sh 脚本；
3. 自动创建`cronjob`定时任务, 每天 0:00 点自动检测所有的证书，如果快过期了，则会自动更新证书。

安装命令执行完毕后，执行`acme.sh --version`确认是否能正常使用`acme.sh`命令。

```bash
cd ~/.acme.sh/
./acme.sh --version
-------------------------------
https://github.com/Neilpang/acme.sh
v2.8.1
```

更高级的安装选项请参考: <https://github.com/Neilpang/acme.sh/wiki/How-to-install>

## 生成证书

`acme.sh` 支持直接使用主流 DNS 提供商的 API 接口来完成域名验证以及一些相关操作具体 [dnsapi 链接](https://github.com/Neilpang/acme.sh/tree/master/dnsapi)

以阿里云为例：

首先获取你的阿里云API Key: <https://ak-console.aliyun.com/#/accesskey>

注：如果使用阿里RAM访问权限控制，需要授权`管理HTTPDNS的权限` `管理云解析(DNS)的权限`这两个权限

编辑acme.sh，加入

```
export Ali_Key="自己的AccessKeyID"
export Ali_Secret="自己的AccessKeySecret"
```
然后执行
```bash
./acme.sh --issue -d 域名 -d *.域名 --dns dns_ali --log
```

`--log`表示生成日志文件`acme.sh.log`，方便生成失败的时候查看原因

如果想自定义证书目录可以加上`-w`参数

## Nginx配置SSL

Nginx支持SSL要先安装ssl模块`--with-http_ssl_module`

生成dhparam

```bash
sudo mkdir /usr/local/nginx/ssl
sudo openssl dhparam -out /usr/local/nginx/ssl/dhparam.pem 2048
```

生成会生成一段时间

### 安装证书

生成的证书是放在`~/acme.sh/域名`下的，这是`acme`内部目录，不应该让nginx直接读取，所以使用`--installcert`命令将证书文件放到其他位置

```bash
./acme.sh  --installcert -d 域名 \
         --key-file /usr/local/nginx/ssl/域名.key \
         --fullchain-file /usr/local/nginx/ssl/fullchain.cer \
         --reloadcmd "service nginx force-reload"
------------或者--------------
./acme.sh --installcert -d 域名 -key-file /usr/local/nginx/ssl/域名.key --fullchain-file /usr/local/nginx/ssl/fullchain.cer --reloadcmd "server nginx force-reload"
```

注：`--reloadcmd`可以不加，这个表示安装完成后执行的命令，但是最好是填上，基本每隔两个月就会执行一次，就需要重载或重启nginx证书才能及时的更新

### 配置Nginx

这里只展示了必要配置

```nginx
server {
    listen      80;
    server_name 域名;
    # 访问80端口重定向到https端口
    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2;
    ssl on;
    server_name  域名;
    ssl_certificate /usr/local/nginx/ssl/fullchain.cer;
    ssl_certificate_key /usr/local/nginx/ssl/域名.key;
    ssl_dhparam /usr/local/nginx/ssl/dhparam.pem;
}
```

ssl的详细设置可以查看[Module ngx_http_ssl_module](http://nginx.org/en/docs/http/ngx_http_ssl_module.html)

