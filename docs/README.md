# SmartDNS

## **教程仅针对Debian系统，其余系统未经测试**

- 此教程分为四部分（篇幅过大，请点击按钮跳转观看）
  - 在服务端，使用SmartDNS，作为系统DNS（例如VPS） [点击此处跳转](https://github.com/jklolixxs/SmartDNS_Rule/blob/main/docs/README.md#%E5%9C%A8%E6%9C%8D%E5%8A%A1%E7%AB%AF%E4%BD%BF%E7%94%A8smartdns%E4%BD%9C%E4%B8%BA%E7%B3%BB%E7%BB%9Fdns%E4%BE%8B%E5%A6%82vps)
  - 在客户端，使用SmartDNS，作为系统DNS（例如OpenWRT） [点击此处跳转](https://github.com/jklolixxs/SmartDNS_Rule/blob/main/docs/README.md#%E5%9C%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8smartdns%E4%BD%9C%E4%B8%BA%E7%B3%BB%E7%BB%9Fdns%E4%BE%8B%E5%A6%82openwrt)
  - 在服务端，使用SmartDNS，与代理软件联动（例如VPS） [点击此处跳转](https://github.com/jklolixxs/SmartDNS_Rule/blob/main/docs/README.md#%E5%9C%A8%E6%9C%8D%E5%8A%A1%E7%AB%AF%E4%BD%BF%E7%94%A8smartdns%E4%B8%8E%E4%BB%A3%E7%90%86%E8%BD%AF%E4%BB%B6%E8%81%94%E5%8A%A8%E4%BE%8B%E5%A6%82vps)
  - 在客户端，使用SmartDNS，与代理软件联动（例如OpenWRT） [点击此处跳转](https://github.com/jklolixxs/SmartDNS_Rule/blob/main/docs/README.md#%E5%9C%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8smartdns%E4%B8%8E%E4%BB%A3%E7%90%86%E8%BD%AF%E4%BB%B6%E8%81%94%E5%8A%A8%E4%BE%8B%E5%A6%82openwrt)

## 在服务端，使用SmartDNS，作为系统DNS（例如VPS）

### 1.安装docker

```
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
```

### 2.选取合适的位置，创建存放配置等文件的文件夹

```
mkdir -p /opt/smartdns && mkdir -p /opt/smartdns/audit && mkdir -p /opt/smartdns/log && mkdir -p /opt/smartdns/cache && touch /opt/smartdns/smartdns.conf && touch /opt/docker-compose.yml
```

（如果您要更改其他目录，请自行将更改后的路径同步到教程后续给的所有路径）

### 3.向smartdns配置文件中写入配置内容

```
curl https://raw.githubusercontent.com/jklolixxs/SmartDNS_Rule/main/docs/server/smartdns.conf > /opt/smartdns/smartdns.conf && curl https://raw.githubusercontent.com/jklolixxs/SmartDNS_Rule/main/docs/server/docker-compose.yml > /opt/docker-compose.yml
```

（教程所给的配置文件已经可以启动使用，但如果您有更进一步的需求，可以查看 /opt/smartdns/smartdns.conf 文件内容，其中有许多指导性注释）

### 4.启动smartdns

```
docker compose -f /opt/docker-compose.yml up -d smartdns
```

### 5.设置为系统DNS

（此处为Debian/Ubuntu设置，其余使用resolv.conf的Linux系统同理）

```
echo nameserver 127.0.0.1> /etc/resolv.conf && chattr +i /etc/resolv.conf
```

`echo nameserver 127.0.0.1> /etc/resolv.conf` ：将smartdns的dns监听地址写入  
`chattr +i /etc/resolv.conf`：将resolv.conf文件锁定，以防某些VPS系统重启后，会自动修改  
如果停止了smartdns，请执行如下命令

```
chattr -i /etc/resolv.conf && echo nameserver 8.8.8.8> /etc/resolv.conf && echo nameserver 8.8.4.4>> /etc/resolv.conf
```

`chattr -i /etc/resolv.conf`：将resolv.conf文件解锁，否则无法编辑  
`echo nameserver 8.8.8.8> /etc/resolv.conf` `echo nameserver 8.8.4.4>> /etc/resolv.conf`：将Google的dns写入，避免系统找不到DNS服务器导致无法接续域名

### 6.验证

```
nslookup bing.com 
```

此时如果设置正确，将会返回类似如下结果

```shell
root@localhost:~# nslookup bing.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
Name:   bing.com
Address: 204.79.197.200
Name:   bing.com
Address: 2620:1ec:c11::200
```

### 7：一些其他相关指令

停止smartdns：

```
docker compose -f /opt/docker-compose.yml down smartdns
```

更新smartdns：

```
docker compose -f /opt/docker-compose.yml pull smartdns
```

查看smartdns日志：

```
docker compose -f /opt/docker-compose.yml logs -f smartdns
```

## 在客户端，使用SmartDNS，作为系统DNS（例如OpenWRT）

### 1.安装docker

```
opkg update && opkg install docker dockerd luci-lib-docker luci-app-dockerman luci-i18n-dockerman-zh-cn docker-compose
```

### 2.选取合适的位置，创建存放配置等文件的文件夹

```
mkdir -p /opt/smartdns && mkdir -p /opt/smartdns/audit && mkdir -p /opt/smartdns/log && mkdir -p /opt/smartdns/cache && mkdir -p /opt/smartdns/rules && touch /opt/smartdns/smartdns.conf && touch /opt/docker-compose.yml
```

（如果您要更改其他目录，请自行将更改后的路径同步到教程后续给的所有路径）

### 3.向smartdns配置文件中写入配置内容

```
curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/docs/client/smartdns.conf > /opt/smartdns/smartdns.conf && curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/docs/client/docker-compose.yml > /opt/docker-compose.yml && curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/blackmatrix7/china-list.conf > /opt/smartdns/rules/china-list.conf && curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/blackmatrix7/global-list.conf > /opt/smartdns/rules/global-list.conf
```

（如果下载不成功，请自行打开代理后，再次执行）  
（教程所给的配置文件已经可以启动使用，但如果您有更进一步的需求，可以查看 /opt/smartdns/smartdns.conf 文件内容，其中有许多指导性注释）

### 4.启动smartdns

```
docker compose -f /opt/docker-compose.yml up -d smartdns
```

### 5.设置为系统DNS

1.进入OpenWRT的Web UI管理页面  
2.找到 **网络**  
3.找到 **DHCP/DNS**  
4.找到 **DNS 转发**  
5.填入 `127.0.0.1#53`  

### 6.验证

```
nslookup bing.com 
```

(OpenWRT固件各式各样，无法保证所有固件都可以执行此命令)  

此时如果设置正确，将会返回类似如下结果

```shell
root@localhost:~# nslookup bing.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
Name:   bing.com
Address: 204.79.197.200
Name:   bing.com
Address: 2620:1ec:c11::200
```

### 7：一些其他相关指令（或是直接在OpenWRT Web UI上直接操作）

停止smartdns：

```
docker compose -f /opt/docker-compose.yml down smartdns
```

更新smartdns：

```
docker compose -f /opt/docker-compose.yml pull smartdns
```

查看smartdns日志：

```
docker compose -f /opt/docker-compose.yml logs -f smartdns
```

## 在服务端，使用SmartDNS，与代理软件联动（例如VPS）

### 1-4步，内容重复，不再赘述 [点击此处跳转查看1-4步](https://github.com/jklolixxs/SmartDNS_Rule/blob/main/docs/README.md#%E5%9C%A8%E6%9C%8D%E5%8A%A1%E7%AB%AF%E4%BD%BF%E7%94%A8smartdns%E4%BD%9C%E4%B8%BA%E7%B3%BB%E7%BB%9Fdns%E4%BE%8B%E5%A6%82vps)

设置代理程序的DNS服务器为smartdns监听地址

- 此处举几个代理软件的例子
  - **sing-box**

      ```json
      {
        ***略***
        "dns": {
          "servers": [
            {
              "tag": "smartdns",
              "address": "udp://127.0.0.1:53"
            }
          ],
          "rules": [],
          "final": "smartdns",
          "disable_cache": false
        },
        ***略***
      }
      ```

  - **xray**

      ```json
      {
        ***略***
        "dns": {
          "servers": [
            "udp+local://127.0.0.1:53"
          ]
        },
        ***略***
      }
      ```

  - **hysteria2**

      ```yaml
      ***略***
      resolver:
        type: udp
        udp:
          addr: 127.0.0.1:53
          timeout: 5s
      ***略***
      ```

## 在客户端，使用SmartDNS，与代理软件联动（例如OpenWRT）

### 1.安装docker

```
opkg update && opkg install docker dockerd luci-lib-docker luci-app-dockerman luci-i18n-dockerman-zh-cn docker-compose
```

### 2.选取合适的位置，创建存放配置等文件的文件夹

```
mkdir -p /opt/smartdns && mkdir -p /opt/smartdns/audit && mkdir -p /opt/smartdns/log && mkdir -p /opt/smartdns/cache && mkdir -p /opt/smartdns/rules && touch /opt/smartdns/smartdns.conf && touch /opt/docker-compose.yml
```

（如果您要更改其他目录，请自行将更改后的路径同步到教程后续给的所有路径）

### 3.向smartdns配置文件中写入配置内容

```
curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/docs/client-proxy/smartdns.conf > /opt/smartdns/smartdns.conf && curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/docs/client-proxy/docker-compose.yml > /opt/docker-compose.yml &&curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/blackmatrix7/china-list.conf > /opt/smartdns/rules/china-list.conf && curl https://testingcf.jsdelivr.net/gh/jklolixxs/SmartDNS_Rule@main/blackmatrix7/global-list.conf > /opt/smartdns/rules/global-list.conf
```

（如果下载不成功，请自行打开代理后，再次执行）  
（教程所给的配置文件已经可以启动使用，但如果您有更进一步的需求，可以查看 /opt/smartdns/smartdns.conf 文件内容，其中有许多指导性注释）

### 4.启动smartdns

```
docker compose -f /opt/docker-compose.yml up -d smartdns
```

设置代理程序的DNS服务器为smartdns监听地址

- 此处举一个例子
  - **openclash**  
    1.进入OpenWRT的Web UI管理页面  
    2.找到 **OpenClash**  
    3.选择 **覆写设置**  
    4.选择 **DNS设置**  
    5.将 **自定义上游 DNS 服务器** 勾选  
    6.向下滚动找到 **设置自定义上游 DNS 服务器**  
    7.点击 **添加**
    8.向 **NameServer** 添加smartdns的监听地址  
    9.向下滚动找到 **设置 SOCKS5/HTTP(S) 认证信息**  
    10.设置用户名与密码，请设置为您喜欢  
    11.选择 **覆写设置**  
    12.选择 **基本设置**  
    13.找到 **HTTP(S)&SOCKS5 混合代理端口**  
    14.设置为您喜欢的端口，或者直接使用其默认端口
    15.更改smartdns配置文件中的代理服务器（vi /opt/smartdns/smartdns.conf）  
    16.重启smartdns（docker compose -f /opt/docker-compose.yml restart smartdns）
    15.OpenClash保存并应用

  - **passwall**  
    由于我并没有用过passwall，就不举他的例子了，各位参考openclash的教程  
