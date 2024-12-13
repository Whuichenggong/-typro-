# 使用ssh远程连接服务器

---



## 1.切换成root用户
sudo …
## 2.启动ssh服务

service ssh star

## 3.检查SSH服务是否启动成功
ps -e| grep ssh 
(出现sshd说明启动成功) 

## 4.启用SSH命令

systemctl enable





一、执行下面的命令安装Nginx

sudo [apt-get](https://zhida.zhihu.com/search?content_id=147538770&content_type=Article&match_order=1&q=apt-get&zhida_source=entity) install nginx

二、初试 Nginx

### 启动 Nginx

执行下面的命令启动Nginx

```text
sudo service nginx start
```

启动之后，我们看一下 Nginx 是否处于运行状态。

```text
sudo service nginx status
```



安装ssh服务

shell

```
apt-get update

sudo apt-get install openssh-client
sudo apt-get install openssh-server
```

启动ssh服务

shell

```
/etc/init.d/ssh start
#显然如下表示启动正常
[] Starting ssh (via systemctl): ssh.service.
```



查看是否安装成功

shell

```
sudo ps -e | grep ssh
```

### 用vim编辑 nginx文件

sudo vim /etc/nginx/nginx.conf

~~~go
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}

 {
       listen 80; # 监听80端口
       server_name 192.168.1.115; # 你的域名或 IP 地址

       root /home/lckfb/html; # 定义静态文件的根目录
       index index.html index.htm; # 默认首页

       location / {
           try_files $uri $uri/ =404; # 尝试访问请求的文件或目录，如果找不到则返回404
       }

       location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
           expires 30d; # 设置静态文件的缓存时间
       }
 }


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#  
~~~



sudo nginx -t 验证nginx 文件

sudo nginx -s reload 重新启动nginx

### 查看网络配置

~~~shell
ipconfig
-bash: ipconfig：未找到命令
lckfb@linux:/usr/sbin$ ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 408  bytes 37654 (37.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 408  bytes 37654 (37.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.115  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::9ee7:4117:2a65:395d  prefixlen 64  scopeid 0x20<link>
        ether 50:41:1c:b4:a8:48  txqueuelen 1000  (以太网)
        RX packets 4847  bytes 1183553 (1.1 MB)
        RX errors 0  dropped 3  overruns 0  frame 0
        TX packets 2517  bytes 310365 (310.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0



ip route
default via 192.168.1.1 dev wlan0 proto dhcp metric 600
169.254.0.0/16 dev wlan0 scope link metric 1000
192.168.1.0/24 dev wlan0 proto kernel scope link src 192.168.1.115 metric 600
~~~





在 Ubuntu 20.04 系统中，您可以通过修改 Netplan 网络配置文件来设置静态 IP。请按以下步骤操作：

1. 通过在终端输入`ip addr`或`ifconfig`命令来识别您的网络接口名称。
2. 创建或编辑 Netplan 配置文件，在终端输入以下命令：



```plaintext
sudo nano /etc/netplan/01-network-manager-all.yaml
```

1. 在打开的配置文件中，进行如下配置：



```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0: # 请以实际的网络接口名称为准
      dhcp4: no
      addresses: [192.168.1.10/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

请确保将其中的网络接口名称、IP 地址、网关和 DNS 服务器地址替换为适合您网络环境的值。

1. 保存并关闭文件。
2. 应用配置，输入以下命令并按提示操作：



```plaintext
sudo netplan apply
```

1. 确认 IP 配置生效：



```plaintext
ip addr
```







## 5.私有ip＋internet



而如果想用私有地址与Internet连接来访问公网，那该怎么做？这就需要将**私有IP地址**转换成**公网IP地址**，与外部连接。

所以，我们平时使用的路由器中会装有一个叫做 **NAT（网络地址转换）** 的软件，我们的路由器中会至少会有一个有效的**公网IP**，**NAT**会将我们的**私有地址**转成路由器中的**公网IP**与外部Internet连接。

而同样的，因为使用的是路由器中的**公共的公网IP**来连接Internet，所以这个内网中的PC在Internet中显示的都是路由器的**公共IP**，这样做不仅提供了一定程度的安全，也可以有效的减缓可用的IP地址空间的枯竭问题。（像我们学校或者公司的内网一般都是这么做的）

另外还有一点，在同一个局域网内，IP地址是唯一的；但是在不同的局域网内，IP地址是可以重复出现的。



## 6.localhost、127.0.0.1和0.0.0.0和本机IP的区别



#### localhost

**localhost**其实是`域名`，一般windows系统默认将**localhost**指向`127.0.0.1`，但是**localhost**并不等于`127.0.0.1`，**localhost**指向的**IP地址**是可以配置的

#### 127.0.0.1

首先我们要先知道一个概念，凡是以`127`开头的**IP地址**，都是**回环地址（Loop back address）**，其所在的回环接口一般被理解为虚拟网卡，并不是真正的路由器接口。

所谓的回环地址，通俗的讲，就是我们在主机上发送给`127`开头的**IP地址**的数据包会被发送的主机自己接收，根本传不出去，外部设备也无法通过回环地址访问到本机。

> **小说明**：正常的`数据包`会从`IP层`进入`链路层`，然后发送到`网络`上；而给`回环地址`发送`数据包`，`数据包`会直接被发送主机的`IP层`获取，后面就没有`链路层`他们啥事了。

而`127.0.0.1`作为`{127}`集合中的一员，当然也是个回环地址。只不过`127.0.0.1`经常被默认配置为**localhost**的**IP地址**。
一般会通过`ping 127.0.0.1`来测试某台机器上的网络设备是否工作正常。

#### 0.0.0.0

首先，`0.0.0.0`是不能被`ping`通的。在服务器中，`0.0.0.0`并不是一个真实的的**IP地址**，它表示本机中所有的**IPV4地址**。监听`0.0.0.0`的端口，就是监听本机中所有**IP**的端口。

#### 本机IP

**本机IP**通常仅指在同一个局域网内，能同时被外部设备访问和本机访问的那些**IP地址**（可能不止一个）。像`127.0.0.1`这种一般是不被当作**本机IP**的。**本机IP**是与具体的网络接口绑定的，比如以太网卡、无线网卡或者PPP/PPPoE拨号网络的虚拟网卡，想要正常工作都要绑定一个地址，否则其他设备就不知道如何访问它。

#### 小例子剧场（帮助理解本机IP、127.0.0.1和0.0.0.0）

现在有两台pc在同一个局域网内，分别为pc1与pc2，pc1上有一个网卡，IP地址为`192.168.10.128`

- pc1中sever监听`127.0.0.1`，则pc1中的client可以连上`127.0.0.1`，`192.168.10.128`连不上；而pc2中client都连不上。
- pc1中sever监听`192.168.10.128`，则pc1中的client可以连上`192.168.10.128`，`127.0.0.1`连不上；而pc2中client能连上`192.168.10.128`。
- pc1中sever监听`0.0.0.0`，则pc1中的client可以连上`127.0.0.1`和`192.168.10.128`，pc2中的client能连上`192.168.10.128`。



## GPT解答



从你描述的情况来看，涉及到的是服务器监听地址和网络接口绑定的问题。我们可以通过分析每种情况来解释为什么会出现这种连接行为。

### 1. **服务器监听 `127.0.0.1`**
   - **服务器**: 当服务器监听 `127.0.0.1` 时，它只会接受来自本机（本地回环接口）的请求。这个地址是指本机的环回网络接口（loopback interface），即服务器的"本地"地址。 
   - **PC1**: 因为 `127.0.0.1` 是本机的回环地址，所以 PC1 上的客户端可以连接到 `127.0.0.1`（即本机）上的服务。
   - **PC2**: 由于 `127.0.0.1` 仅限于本机通信，`192.168.10.128` 这个 IP 地址属于不同的设备（PC2），因此 PC2 无法访问 `127.0.0.1`（PC1的本机地址）。

   **总结**: `127.0.0.1` 只能用于本机通信，其他设备无法访问。

### 2. **服务器监听 `192.168.10.128`**
   - **服务器**: 当服务器监听 `192.168.10.128`（PC1的局域网 IP 地址）时，它只会接受来自该 IP 地址上（或者在同一子网中的）客户端的连接。
   - **PC1**: 本机的客户端可以连接到 `192.168.10.128`，因为它正在监听该地址。
   - **PC2**: 由于 `192.168.10.128` 是在同一局域网中的地址，PC2 能够通过局域网访问这个 IP 地址上的服务。
   - **PC1（本机）连接 `127.0.0.1`**: 在此情况下，由于服务器绑定到 `192.168.10.128`，本机无法连接到 `127.0.0.1`（回环地址），因为服务不再监听回环接口，而是监听局域网地址。

   **总结**: 只要客户端与服务器在同一局域网中，它们就可以通过 `192.168.10.128` 来通信，但本机不能访问 `127.0.0.1`，因为服务只绑定到局域网 IP。

### 3. **服务器监听 `0.0.0.0`**
   - **服务器**: 当服务器监听 `0.0.0.0` 时，它会监听所有可用的网络接口，意味着它可以接受来自本机和其他设备的连接。这包括本机的回环接口（`127.0.0.1`）以及本机的局域网接口（例如 `192.168.10.128`）。
   - **PC1**: 本机客户端可以通过 `127.0.0.1` 或 `192.168.10.128` 连接到服务器，因为服务器同时绑定到了回环地址和局域网地址。
   - **PC2**: 由于服务器监听 `0.0.0.0`，它不仅能接受来自本机的连接，还能接受来自其他机器（如 PC2）的连接。因此，PC2 可以连接到 `192.168.10.128`（PC1的局域网 IP 地址）。

   **总结**: `0.0.0.0` 使服务器接受所有接口上的连接，包括本机和其他机器的连接。

### 关键点总结：
- **`127.0.0.1`**: 仅能接收来自本机的连接，其他机器无法访问。
- **`192.168.10.128`**: 只接收来自本机和同一局域网内其他设备的连接。其他局域网设备可以访问，但本机不能访问回环接口。
- **`0.0.0.0`**: 接收来自所有网络接口（包括本机和其他设备）的连接，适用于需要外部设备连接的情况。

### 解决方案：
如果你希望所有设备（包括本机和其他机器）都能访问服务器服务，你应该配置服务器监听 `0.0.0.0`。这样，服务器就可以同时接受来自本机和其他设备的连接，确保所有设备都能通过指定的 IP 地址连接。

### 为什么 PC2 不能连接到 `127.0.0.1`？
`127.0.0.1` 是本机的回环地址，仅允许本机访问。如果你在 `PC1` 上将服务器绑定到 `127.0.0.1`，那么只有 `PC1` 能连接到该地址，`PC2` 无法访问，因为 `127.0.0.1` 是本地回环接口，外部机器无法访问该地址。
