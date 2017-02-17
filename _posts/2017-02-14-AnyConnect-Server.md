---
layout: post
title: "搭建一个OpenConnect VPN服务器"
date: 2017-02-14 16:44:00
categories: coding
---

之前在linode上搭了一个shadowsocks，容易被封，加上一些网络原因，不是很顺手，所以想换个VPN用。
朋友推荐，VULTR就不错，节点多，价格不高。选了个JP的节点，装完机可以搞起。

VULTR上就有文档教你怎么装
[文档](https://www.vultr.com/docs/setup-openconnect-vpn-server-for-cisco-anyconnect-on-ubuntu-14-04-x64)
步骤大致上是这样，但例子里面ubuntu和ocserv的版本都比较老，
我用的是ubuntu16.10，现在官方的长期支持版本，装东西方便，
ocserv也更新到了0.11.6。
可能是因为版本不同，遇到几个依赖问题，所以稍微把文档翻译并补充一下。
在文档之外，还有一些优化的东西，也一并记录下来

### 安装ocserv

先下一个ocserv

	wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.11.6.tar.xz
	tar -xf ocserv-0.11.6.tar.xz
	cd ocserv-0.11.6

文档里的版本比较老，可以下个最新的，我下的是0.11.6

装一些依赖

	apt-get install build-essential pkg-config libgnutls28-dev libwrap0-dev libpam0g-dev libseccomp-dev libreadline-dev libnl-route-3-dev

文档上的依赖只有这些，但在后续执行configure的时候报错缺少libev4，如果遇到可以再装一个libev-dev,
make的时候还遇到一个报错缺少autogen，安装autogen之后重新执行configure，然后再make即可

	apt-get install libev-dev autogen

linux安装三板斧

	./configure
	make
	make install

生成CA证书和服务器证书

	cd ~
	apt-get install gnutls-bin
	mkdir certificates
	cd certificates

创建一个CA证书模板文件ca.tmpl

	cn = "VPN CA" 
	organization = "Big Corp" 
	serial = 1 
	expiration_days = 3650
	ca 
	signing_key 
	cert_signing_key 
	crl_signing_key 

cn和organization岁自己喜欢写

生成CA密钥和证书

	certtool --generate-privkey --outfile ca-key.pem
	certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem

这时候当前目录下会多出来ca-key.pem和ca-cert.pem两个文件

创建一个服务器证书模板文件server.tmpl，
这里cn字段是你服务器的域名或者是IP地址，
这里cn字段是你服务器的域名或者是IP地址，
这里cn字段是你服务器的域名或者是IP地址

	cn = "you domain name or ip"
	organization = "MyCompany" 
	expiration_days = 3650 
	signing_key 
	encryption_key
	tls_www_server

然后用刚刚生成的CA密钥和CA证书，生成服务器的key和证书

	certtool --generate-privkey --outfile server-key.pem
	certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem

同样，目录下多出来两个文件server-key.pem和server-cert.pem

在/etc下建一个ocserv目录，把服务器证书和密钥文件拷贝过去，ocserv里面的doc目录下有一份默认配置可以做参考，我们的config文件就用这个做基础改

	mkdir /etc/ocserv
	cp server-cert.pem server-key.pem /etc/ocserv
	cd ~/ocserv-0.11.6/doc
	cp sample.config /etc/ocserv/config
	cd /etc/ocserv

修改config里面几行

	auth = "plain[/etc/ocserv/ocpasswd]"		登陆验证方式

	try-mtu-discovery = true 					开启mtu

	server-cert = /etc/ocserv/server-cert.pem 	指定证书和密钥
	server-key = /etc/ocserv/server-key.pem

	dns = 8.8.8.8 								google的DNS，喜欢用别的也可以

	# comment out all route fields 				这些用不上，注释掉
	#route = 10.10.10.0/255.255.255.0
	#route = 192.168.0.0/255.255.0.0
	#route = fef4:db8:1000:1001::/64
	#no-route = 192.168.5.0/255.255.255.0

	cisco-client-compat = true 					开启思科AnyConnect客户端兼容

文档上说改这些，但如果你平时工作有公司内网之类的，也可以把他们设成不走VPN。
举个例子，我就把局域网段都屏蔽掉了。

	no-route = 192.168.5.0/255.255.255.0
	no-route = 172.16.0.0/255.240.0.0
	no-route = 10.0.0.0/255.0.0.0

创建一个用户试试，username就是用户名，完事儿会让你输入密码。账号密码就存在ocpasswd文件里，就是刚刚在config里面指定的文件

	ocpasswd -c /etc/ocserv/ocpasswd username

防火墙把NAT转换开启

	iptables -t nat -A POSTROUTING -j MASQUERADE

开启ipv4转发，编辑/etc/sysctl.conf

	net.ipv4.ip_forward=1

改完要绑定一下才能生效

	sysctl -p /etc/sysctl.conf

都完事儿之后就可以启动ocserv了

	ocserv -c /etc/ocserv/config

在电脑，或者手机下思科的AnyConnect客户端，用刚刚创建的用户名和密码连接就可以了。
因为证书是我们自己签的，会提示不让连接不受信任的服务器，所以要在设置里把这个选项关掉。

### 把ocserv包装成服务

包装成服务一是比较稳定，二是启动停止比较方便，不用手动杀进程

ubuntu 16已经放弃了upstart，改用systemd来做进程守护的工作。
网上的文章多数还停留在ubuntu 14，用的upstart脚本，没找到使用systemd的参考
所以只好自己动手摸索一下了，这里记录一下使用systemd怎么配置

同样在ocserv的doc目录下，有默认的systemd配置，我们在这个配置基础上做修改

	cp ~/ocserv-0.11.6/doc/systemd/standalone/ocserv.service /lib/systemd/system
	vim /lib/systemd/system/ocserv.service

修改启动命令

	- ExecStart=/usr/sbin/ocserv --foreground --pid-file /var/run/ocserv.pid --config /etc/ocserv/ocserv.conf
	+ ExecStart=/usr/local/sbin/ocserv --foreground --pid-file /var/run/ocserv.pid --config /etc/ocserv/config

可以用which ocserv看看ocserv装在什么位置，我的在/usr/local/sbin/ocserv，对应修改。
然后配置文件指定到之前放的位置/etc/ocserv/config

保存之后重新加载配置

	systemctl daemon-reload

接着就可以使用命令启动，停止，重启ocserv服务了

	systemctl start/stop/restart ocserv

通过status可以查询当前服务的状态

	systemctl status ocserv

更多关于systemd的内容，可以参考阮博士的博客
[Systemd 入门教程](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

### 域名与证书

因为CA证书是自签的，AnyConnect连接的时候会提示是不受信任的服务器。
恰好自己想搞个域名耍耍，有了域名就可以给服务器签一个受信任的证书了。

首先，需要买一个域名，货比三家之后发现namecheap比较便宜，站点做得也比较好看，就它了。
注册完域名，设置A record，指到自己的服务器的IP。（为了完成后续的验证，把www的record也加上）

let\`s encrypt是一个免费的CA，但需要向他证明域名在自己的掌控下，官方推荐的验证方法是使用certbot进行验证。
具体来说就是在服务器上跑一个客户端，会临时启一个web服务，let\`s encrypt访问你的域名，就完成了验证过程。
需要注意的是，如果服务器上80端口自己有跑web服务，会发生冲突。

先安装certbot客户端

	apt-get install certbot

然后进行验证

	certbot certonly --standalone -d example.com -d www.example.com

会有输入邮箱等步骤，完成验证之后会把证书放在/etc/letsencrypt/live/example/下面(example就是你的域名)

接着我们就可以把之自己签的CA证书和服务器证书删除，改用let\`s encrypt签发的证书。

	cd /etc/ocserv
	ln -s /etc/letsencrypt/live/example/fullchain.pem server-cert.pem
	ln -s /etc/letsencrypt/live/example/privkey.pem server-key.pem
	systemctl restart ocserv

因为let\`s encrypt签的证书时间只有90天，所以到期之前需要续签。
可以直接用定时任务重新跑一边certbot，也可以交给certbot做这件事

	certbot renew --dry-run 
