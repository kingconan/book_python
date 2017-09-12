## 内网穿透

需要的东西

* 服务器，有公网IP
* 内网机器，有内网固定ip

操作步骤

1. 服务器上，根据系统下载对应版本的frp文件

```bash
#ucloud hk ubuntu
https://github.com/fatedier/frp/releases/download/v0.12.0/frp_0.12.0_linux_amd64.tar.gz
#解压
tar -zxvf frp_0.12.0_linux_amd64.tar.gz
#删除客户端相关文件，只保留服务器端的就好
rm -f frpc frpc_full.ini frpc.ini
#编辑配置文件
[common]
bind_port = 7000
vhost_http_port = 8082
dashboard_port = 7500
dashboard_user = kingconan
dashboard_pwd = 5040309511
max_pool_count = 5
authentication_timeout = 900

[ssh]
listen_port = 6000
auth_token = kingconan_mac
#启动
./frps -c frps.ini
```

2. 客户端，根据系统版本下载对应frp文件

```
#mac对应的是frp_0.12.0_darwin_amd64
#删除服务器相关文件
rm -f frps frps_full.ini frps.ini
#编辑配置文件
[common]
server_addr = 45.249.247.123
server_port = 7000
auth_token = kingconan_mac
pool_count = 1

[ssh]
type = tcp
local_ip = 192.168.1.30
local_port = 22
remote_port = 6000


[web]
type = http
local_port = 80
custom_domains = 45.249.247.123
#启动
./frpc -c frpc.ini
#此时可以服务器和客户端可以看到相关信息

#客户端
2017/09/12 13:43:57 [I] [control.go:255] [0c2ee4505d0c8758] login to server success, get run id [0c2ee4505d0c8758]
2017/09/12 13:43:57 [I] [control.go:367] [0c2ee4505d0c8758] [ssh] start proxy success
2017/09/12 13:43:57 [I] [control.go:367] [0c2ee4505d0c8758] [web] start proxy success
#服务器端
2017/09/12 13:43:57 [I] [service.go:212] client login info: ip [116.226.240.122:55747] version [0.12.0] hostname [] os [darwin] arch [amd64]
2017/09/12 13:43:57 [I] [proxy.go:165] [0c2ee4505d0c8758] [ssh] tcp proxy listen port [6000]
2017/09/12 13:43:57 [I] [control.go:318] [0c2ee4505d0c8758] new proxy [ssh] success
2017/09/12 13:43:57 [I] [proxy.go:204] [0c2ee4505d0c8758] [web] http proxy listen for host [45.249.247.123] location []
2017/09/12 13:43:57 [I] [control.go:318] [0c2ee4505d0c8758] new proxy [web] success
```

3. 访问公网IP `45.249.247.123:8082` 可以穿透到内网对应IP，本例中到了192.168.1.30的80端口对应的我的测试travelid网站



## 参考链接

 https://segmentfault.com/a/1190000009895225



