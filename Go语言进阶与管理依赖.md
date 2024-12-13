# 字节青训营学习





### 一.入门篇学习

实战篇： 

#### 1.猜数字游戏

运用 

~~~
”math/rand“

maxNum := 100
调用 Intn（maxNum）
//注意这并不能使每一次都用都产生不同的值

需要加 时间戳 才能每次产生不同的值
应该是：
maxNum := 100
ran.Seed(time.Now().UnixNano())
然后调用
 ran.Intn（maxNum）
 
 用 "bufio"
 这个特别的包以后可以用到
 
 这里读取一行输入用的是
 reader := bufo.NewReader(os.Stdin)//调用这个可以更加灵活
 input， err := reader.ReadString('\n')
 if err != nil{...}
 
 input = strings.TrimSuffix(intput,"\n")//去掉换行符
 
 guess,err := strconv.Atoi(input)//转换成数字
 
 菜值逻辑 
~~~



#### 2.在线词典介绍



~~~
go run simpledict/v4/main.go hello
//意思是查询hello这个功能
//会输出以下内容
hello UK:['he'lau]US:[ha'lo]
int.喂；哈罗
n.引人注意的呼声
V.向人呼（喂

调用第三方api

~~~



生成请求

https://curlconverter.com/

写入curl 自动生成代码

示例：

![image-20241104201754858](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241104201754858.png)



会输出一些列bilibili的东西





json序列化

![image-20241104202957339](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241104202957339.png)

衍生出以下

![image-20241104203014855](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241104203014855.png)







~~~
结构体
~~~



~~~
request ：= DicRequest{Trans Type："en2zh",Source:"good"}
buf, err := json.Marshal(request)//序列化request 变成byte数组
if~~~
var data = bytes.NewReader(buf) //因为buf返回的是bytes数组所以我们应该 bytes.NewReader
~~~



解析response 进行反序列化



json转golang 结构体

https://oktools.net/json2go

反序列化

~~~
err = json.Unmarshal(bodytext,&dictResponse)//传入结构体

fmt.Println("%#v",dicResponse)


~~~





### 二.Go进阶

#### 1.并发编程

并发：多线程在一个核运行 时间碎片

并行： 多核



![image-20241109132249854](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109132249854.png)



并行是并发的手段





#### 2.协程



![image-20241109132406297](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109132406297.png)



协程：轻量级线程 线程本身重量级 Goroutine：可以实现上万个携程

例子:协程

![image-20241109132554426](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109132554426.png)



go关键字开启协程



通过通信来共享内存



![image-20241109132746410](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109132746410.png)

channel

make创建channel

![image-20241109132925436](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109132925436.png)

有缓冲：

无缓冲：进行通信时（两个goroutine同步）也称同步通道



示例：

![image-20241109133236164](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109133236164.png)

先make创建通道 把数字放入第一个通道里

然后b把src做平方运算



并发安全 Lock

![image-20241109134052647](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109134052647.png)

Lock（）临界区   



![image-20241109134621688](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109134621688.png)





#### 3.依赖管理

1.GOPATH

bin pkg src（项目源码）

无法实现package 的多版本控制

2.vendor存放依赖副本 也有弊端

3.go module 管理 解决了问题 



![image-20241109135629389](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109135629389.png)



indirect关键词



![image-20241109140344112](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241109140344112.png)

b（选择最低兼容版本）



#### 4.依赖分发

Github         SVN               …

  

​          Developer

无法保证构建稳定性

无法保证依赖可用性

增加第三方压力



Go proxy （）缓存内容版本 从proxy拉取依赖 减少第三方压力



Proxy1 -》 proxy2 -》 Direct  依次进行



#### 5.工具 go get 



go mod       **init tidy download** 



#### 6.测试

事故：

![image-20241111092419264](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111092419264.png)

测试的重要性：避免事故的最后一道屏障

单元测试 mock测试   回归测试 集成测试



##### 1.单元测试



![image-20241111092633595](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111092633595.png)

保证代码整体覆盖率 

提升效率



##### 2. 规则



![image-20241111092852792](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111092852792.png)



##### 3例子：



![image-20241111093035601](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093035601.png)



go test 【flags】 【packages】



##### 4.assert



![image-20241111093202556](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093202556.png)



##### 5.覆盖率

![image-20241111093237237](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093237237.png)



![image-20241111093252039](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093252039.png)



提升覆盖率



![image-20241111093436343](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093436343.png)

对各个分支测试 使函数代码都经过完备的测试 提升覆盖率 减少事故



![image-20241111093526971](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093526971.png)



##### 6.依赖



![image-20241111093723869](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093723869.png)



##### 7.文件处理



![image-20241111093855592](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111093855592.png)



一旦文件被别人篡改 在特定场景下就无法运行！



##### 8.Mock

![image-20241111094015817](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111094015817.png)



replacement 打桩函数



![image-20241111094459861](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111094459861.png)



通过mock 不对 文件有强依赖

defer 大打桩函数的卸载



##### 9.基准测试



![image-20241111094657452](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111094657452.png)



![image-20241111094758462](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111094758462.png)



优化：



![image-20241111094944759](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111094944759.png)



fastrand 



#### 10.项目实践



![image-20241111095222107](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111095222107.png)



![image-20241111095416063](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111095416063.png)





![image-20241111095526585](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111095526585.png)



![image-20241111100016448](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111100016448.png)



话题id -》 获取所有post



![image-20241111100134049](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111100134049.png)



![image-20241111100208797](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111100208797.png)

初始化话题内存索引



![image-20241111100306256](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111100306256.png)



---

逻辑层：

![image-20241111100415573](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111100415573.png)



![image-20241111121526603](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111121526603.png)



![image-20241111121719613](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111121719613.png)



并行处理

![image-20241111121901723](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111121901723.png)



![image-20241111122045428](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241111122045428.png)





### 三.高质量编程与实践

如何编写更简洁和跟清晰的代码

熟悉GO程序性能分析工具

常用Go语言程序优化手段

了解工程性能优化的原则和流程

:性能调优    性能分析工具     性能调优案例

算法效率



#### 1.高质量编程

正确可靠 简洁清晰

各种边界条件是否考虑完备

异常情况处理 稳定性保证 

易读易维护

团队合作保证容易读 维护 使其增加和调整更加快速 更加清晰

主要是给人看 让人可以看懂  对已有的功能改善 优化 容易添加功能 



#### 2.编码规范

代码格式

注释

命名规范

控制流程

错误和异常处理



![image-20241112171014037](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112171014037.png)



不需要注释实现接口的方法 这种注释可以删除

![image-20241112171126665](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112171126665.png)



##### 1.gofmt

推荐使用gofmt 自动格式化代码为官方统一风格



##### 2.注释

 注释应该解释代码的作用

注释应该解释代码如何做的

注释应该解释代码实现的原因

注释应该解释代码什么情况会错



![image-20241112171511567](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112171511567.png)

下面的根本不需要注释没有什么必要 函数名字已经说明了



![image-20241112171655993](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112171655993.png)

最后一条语句是很难理解的  如果没有注释   一定要会看英文呵呵



![image-20241112171909248](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112171909248.png)

  

![image-20241112172349471](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112172349471.png)



![image-20241112172623038](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112172623038.png)



![image-20241112172638016](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112172638016.png)

此时的t就减少了很多东西 



![image-20241112172953450](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112172953450.png)

例如http中调用 Server 是 http.Server

 若用 ServerHTTP   http.ServerHTTP这样感觉就变得冗余了没有必要





![image-20241112173231246](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173231246.png)



避免if else嵌套 包含同样语句可以去掉 重复语句

![image-20241112173423686](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173423686.png)



这样看起来比较复杂

![image-20241112173532576](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173532576.png)

调整后：

![image-20241112173605956](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173605956.png)



控制流程 ： 线性原理 尽量走直线 避免复杂的嵌套分支

##### 3.错误和异常处理

![image-20241112173840011](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173840011.png)



![image-20241112173902743](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112173902743.png)



![image-20241112174134842](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241112174134842.png)





#### 3.性能优化



##### GO语言优化

内存管理优化

编译器优化



什么是性能优化 ，为什么要做性能优化？

：提升软件系统处理能力，减少不必要的消耗

：用户体验，让用户刷抖音不卡顿

：资源高效利用，成低成本，提高效率  



业务代码：处理用户请i去

SDK：go的SDK 

基础库：

这两部分提供抽象逻辑（数据结构 网络库 io库）



语言运行时： gc 调度器（go语言）

OS



性能分析工具：pprof

依靠数据而非猜测

优化最大瓶颈



Go的SDK

接口 命令 APIs

测试 来 驱动开发

隔离： 通过选择控制是否开启优化

可观测：

静态分析：



##### 自动内存管理

基于追踪的垃圾回收

GC



动态内存 malloc

自动内存回收： 避免手动内存管理 专注于实现业务逻辑

保证内存使用的正确性和安全性

为新对象分配空间

![image-20241120170653872](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241120170653872.png)



 





































####  四.数据库



##### 数据库database/sql



![联想截图_20241121181508](C:\Users\30413\Pictures\联想截图\联想截图_20241121181508.png)





![image-20241121181911544](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241121181911544.png)



GORM ： 业务需求驱动开发



![image-20241121182351306](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241121182351306.png)



**基本用法**

![image-20241121182409368](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241121182409368.png)



![image-20241121182623399](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241121182623399.png)



**规定**



![联想截图_20241121182853](C:\Users\30413\Pictures\联想截图\联想截图_20241121182853.png)



数据库约束 

Select实现级联删除







#### .数据结构与算法

##### 1.经典排序算法

为什么要用数据结构和算法：







#### 五.网络与部署



《负载均衡 高并发网关原理与实践》

协议基础 

协议分析（自学）

熟悉tcp/ip 熟悉计算机网络



抖音视频 加载出来 会有什么交互（网络是如何交互的）（为什么刷抖音又快又稳）（计算机网络要解决什么问题 发现什么问题）



应用层：  域名解析（DNS） 图片下载 视频下载（HTTP）评论API/HTTP 



网络接入 网络传输

手机要先访问抖音服务器



![image-20241116152733233](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116152733233.png)



终端（pc pad） 通过 4g /wifi 通过有线网络 接入 四大运营商网络 -》 接通抖音的机房（服务器） 还用通过光缆（海底） 接通 美国网络



##### 网络接入 （路由）



交换机/逻辑交换机/网络虚拟化



路由一定对称吗？



路由是工作在哪一层协议？

路由协议（ip层） 本身不是ip层 动态路由协议（传输层协议bgp）（基于tcp udp）可能 不是很简单



路由改的是ip地址吗？



不是改ip地址 而是mac地址 路由是为了找到目标ip  

怎么找下一跳（网络中间节点）的MAC 通过 ARP协议 跨网段 不能发送ARP  

同网段可以发送ARP 否则需要一级一级的发送  免费ARP：（）新加入机器（向其中发送免费ARP ： 防止ip冲突 在同一局域网里有两个同一个ip ） 



ARP：本质是查找下一跳MAC 而不是目标请求地址

ARP 代理： 中间设备抢先应答 （）



![image-20241116154311845](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116154311845.png)





![image-20241116154131510](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116154131510.png)



IP协议 

唯一标识 

为什么Mac（2层）（以太网） 地址不能代替ip协议 ： IP协议把MAC地址问题 解决了



ipv4不够用 怎么解决：如果不支持ipv6 ， 用NAT（原理 内部用户 通过NAT） 



![image-20241116155224812](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116155224812.png)



问题2 ： （NAT ip＋端口）一起改变 解决第二个问题



##### 网络打通视频怎么下载



![image-20241116155401344](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116155401344.png)



![image-20241116155543012](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116155543012.png)



也就是服务的封装与拆解         



如何把域名映射到ip （DNS基于UDP协议（端口＋校验））



![image-20241116155905523](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116155905523.png)



避免分片 

丢包丢了UDP也不知道



##### TCP连接 三次握手



TCP连接状态

如果拔了网线，连接会断吗（没有什么关系）

keep-alive ：保活机制（不会一定断开在一定场景下）



##### TCP传输



![image-20241116160324196](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116160324196.png) 



很复杂的协议 

Timewait？（） 状态复杂

TCP丢包

滑动窗口

流量控制



##### 网络传输 HTTP/HTTP1.1



![image-20241116161839691](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116161839691.png)



HTTPS 解密 出来仍然是HTTP

防止中间者偷听一些东西  加密之后中间人听不懂



SSL/TLS握手 非对称加密 对称加密



![image-20241116162211539](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241116162211539.png)



##### 网络架构



网络提速

HTTP 2.0 （多路复用） 并行下载 并行访问

一次性加载了多个图片 （并行请求 一次性发送了）

![image-20241117194455726](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117194455726.png)



多路复用： 



![image-20241117194604452](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117194604452.png)



TCP （丢包 其他包等待（对头阻塞））



![image-20241117194853572](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117194853572.png)



**协议优化：**

TCP本身协议不可插拔

UDP（基于UDP扩展） 

kernel （windows mac 安卓 ios ）是否都要去实现呢？

Google实现在了用户态（方便）

RTT

QUIC（实现了UDP的扩展）(弱网优势) （解决了队头堵塞）（优化了HTTp2.0的多路复用）



**路径优化**



![image-20241117195324763](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117195324763.png)

服务器集合（数据中心）



POP接入（核心机房） 边缘机房（靠近用户（上海电信 上海移动……））



**同运营商访问** （访问客户端ip  电信 解析到 电信）

电信 访问 电信

移动 访问 移动

若要使两者跨网访问（丢包率较高） 



路径优化（CDN） 网络提速-静态资源 （边缘机房（缓存）直接从缓存中取出资源 如果找不到 -》 核心机房）



动态API （播放 / 评论接口）（因为信息不一样）

![image-20241117195852655](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117195852655.png)

DSA（路径优化算法） 直连最快 

机房延迟探测 （做成表（通过算法 找到最优路径））



##### 网络提速的优化之路

几天就挂了怎么办？？



抖音稳定性如何调高？



![image-20241117200255063](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117200255063.png)



![image-20241117200401223](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117200401223.png)



专线： 内部字节机房-》没有走外部internet（如果走internet 如果从北京 访问 上海 可能会造成中间去到江苏再回到上海这样会很慢 并且丢包等）-》自己拉线连接两个机房（通过交换机等）（这样速度更快）  

b -》 c  外网：机房内部专线以外的网络通过internet连接 （如果专线挂了）需要走外网容灾 微服务可能跨机房



![image-20241117200824142](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117200824142.png)



全局容灾系统 ：  A机房不可用了 （自动容灾） 探测机房b的容量可以承载 a机房的容量

自动降级/容灾

云到端 SDK告诉端，不要访问，崩了的A机房   云控   什么场景云控控制不到（Web服务器 字节搜索/百度搜索 ）

 

故障明确

沟通： 明确是什么业务 什么接口故障

故障体现在哪？ 其他目标是否访问正常（业务A有故障 接口A有问题 其他的是否有问题？）

是否是修改导致的异常？（如果是就回退）

如果你上传导致的错误（找是谁 ）



先止损 再 排查（debug）

如何止损 



分段排查



![image-20241117202312571](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117202312571.png)





##### 常用指令



**![image-20241117202359378](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117202359378.png)**



最后一个是抓包工具



误判断 摘除了 好的服务器

线上debug（p0级别事故）

![image-20241117202838541](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117202838541.png)





抓包：

快速发包（路由对称）实际路由并不是对称的（找下一跳） 故障预防真的很重要 



##### 课后作业

![image-20241117203250586](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117203250586.png)





![image-20241117203354461](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241117203354461.png)



#### 企业级



输入网页到内容加载出来 中间都经历了什么 （TCP握手 SSL 域名解析）

浏览器抓包

看第一条请求了什么

![image-20241118125748346](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118125748346.png)





网络卡 还是服务器满 还是什么？



企业接入：



域名系统 自建DNS服务器 HTTPS 接入全站加速 四层负载均衡 七层负载均衡



example公司  Host -》 ip映射



问题： 流量和负载 名称冲突 时效性（）



##### 使用域名系统

使用域名系统 替换hosts



![image-20241118130222229](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118130222229.png)



域名购买  购买二级域名 example.com  域名备案防止从事非法运动



建设外部网站：

![image-20241118130420184](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118130420184.png)



自建DNS：

取代云厂商



![image-20241118130700240](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118130700240.png)

先访问 本地DNS服务器 -》 根 -》 顶级 -》权威-》本地缓存

##### DNS查询过程



![image-20241118130854468](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118130854468.png)



##### 权威DNS系统架构

![image-20241118131553232](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118131553232.png)



##### 接入HTTPS协议

HTTP明文传输 被抓去后 信息很容易被暴露出来



加密算法：

对称加密： 一份密钥



非对称加密 ： 公钥和私钥 （公钥加密私钥解密 或者  对调）（锁头和钥匙）

 

SSL 通信过程：



![image-20241118174903171](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118174903171.png)



证书链：

公钥是不是可信的 会不会被劫持？

数字签名



外网访问站点一定是一帆风顺的吗：

1.源站（网站） 容量第 可承载的并发请求数低 容易打跨

2.报文经过的网络设备越多 出问题概率越大 丢包 劫持

3.自主选择网络链路长 时延高  

整体看来：就是响应慢 卡顿



优化：

增加后端机器扩容 静态内容

全站加速：

静态加速：CDN 

cpu访问 （缓存）



动态加速： 



![image-20241118175913629](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118175913629.png)



DCDN原理 ： 

RTT： 用户到核心

用户到边缘

边缘到汇聚

汇聚到核心



##### 全站加速



![image-20241118180340066](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118180340066.png)





![image-20241118180356658](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241118180356658.png)





#### 消息队列 kafka



用户行为： 搜索 点赞 评论 收藏

使用场景： 搜索服务， 直播服务 订单服务 支付服务



如何使用Kafka： 创建集群 新增topic 编写生产者逻辑 编写消费之逻辑



![image-20241125172137241](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125172137241.png)





![image-20241125172850441](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125172850441.png)



Kafka架构：



Zookeeper



Producer 批量发送

  		数据压缩

Broker 数据存储 消息文件结构



磁盘结构

​		操作系统：![image-20241125174036284](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125174036284.png)					



顺序写： 提高写入效率 



![image-20241125174650570](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125174650570.png)



Broker 偏移量索引文件

二分查找

![image-20241125174726467](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125174726467.png)





Broker 时间戳索引文件



传统数据拷贝：



操作系统层面： 数据的内存拷贝（开销很大）

![image-20241125175012153](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125175012153.png)



Broker零拷贝



![image-20241125175137912](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125175137912.png)

 

Consumer-消息接收端





![image-20241125175549378](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125175549378.png)



通过手动分配 哪一个Consumer消费哪一个Partition 完全由业务决定



缺点

不能自动容灾 

优点：

快



自动分配 High-level



![image-20241125180033073](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125180033073.png)









![image-20241125180233648](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241125180233648.png)



消息队列BMQ

秒级完成 比Kafak更快





#### 存储与数据库



数据的持久化



1.校验数据的合法行

（名字是否存在） （修改内存 用高效的数据结构组织数据）（写入存储介质 以寿命 性能友好写入硬件）（）



![image-20241126175430987](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241126175430987.png)



什么是存储系统？

一个提供了 读写 控制类接口 能够安全有效的 把 数据持久化的软件 称为 存储系统

user Medium Memory Network(把原有的单机 升级到分布式系统) 还可能与这些有关



特点：

性能敏感 容易受硬件影响  代码既简单又复杂（考虑到多种异常情况）



存储器层级结构：

Persistent Memory

数据怎么从应用到存储介质



![image-20241126180440273](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241126180440273.png)



数据拷贝 消耗cpu 如果cpu全部用来拷贝 软件性能就会降低（减少拷贝的使用） Disk（）



RAID技术： 单机存储怎么做到 高性能/高性价比/高可靠性

背景： 大容量磁盘价格》 多块小容量磁盘

单块磁盘的写入性能 《 多块磁盘的并发写入性能



RAID0

 多块磁盘的简单组合 

数据条带化存储 调高磁盘带宽

没有额外的容错设计

RAID1

一块磁盘对应一块额外镜像盘

真是空间利用率50%

容错能力强



上面是两个极端



RAID 0 + 1

RAID 0 和 RAID 1



##### 数据库和存储系统不一样吗？

关系型数据库（是存储系统） 

非关系型数据库



关系是什么： 关系模型（EFCodd）= 集合 反映了事务间的关系

关系代数 = 运算的抽象查询语句

SQL = 一种DSL （方便人类阅读的 关系代数表达式）



关系型数据库：结构化数据友好 支持事务（ACID）支持复杂的查询语言（sql 全集 子集）

非关系型数据库（也是存储系统）： 半结构化数据友好 可能支持事务（ACID） 可能支持复杂查询语言



数据库 vs 经典存储 - 结构化数据管理

![image-20241126181756913](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241126181756913.png)



##### 事务能力：

要么全做 要么全不做

数据状态是一致的

可以隔离多个并发事务 避免影响

事务一旦提交成功 数据保持持久性



#####  复杂查询能力：

复杂查询： 请查询出以xiao开头 密码提示问题小于10个字的人 并按照性别分组统计人数



![image-20241126182318775](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241126182318775.png)

左边灵活简介 右边复杂



#### 支流产品剖析



##### 单机存储系统

单个计算机节点上的存储软件系统 一般不涉及网络交互

key-value存储 本地文件系统



本地文件系统

一切皆文件：

文件系统管理单元： 文件

文件系统接口： 文件系统繁多 Ext2 遵循VFS统一抽象接口



linux 文件系统的两大数据结构 ：Index Node   Directory Entry



1. innode 记录文件元数据 如id 大小 权限 磁盘位置
2. innode 是一个文件的唯一标识  会被存储到磁盘上
3. innode总数在格式化文件系统时就固定了

​	1：1



1.Directory Entry

记录文件名 innode指针 层级关系 

dentry时内存结构 与innode 关系时N：1



key-value存储

![image-20241126183518576](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241126183518576.png)



put(k,v) get(k)

LSM-Tree 某种程度上牺牲读性能 追求写入性能（）

拳头产品： RocksDB



分布式存储：

在单机存储基础上实现了分布式协议，实际大量网络交互

分布式文件系统  分布式对象存储系统

HDFS ：

核心：

 支持海量数据存储（使用普通硬件堆叠）

高容错性

弱POSIX语义

使用普通x86服务器-极高性价比高

DataNode



Ceph- （分布式存储）开源分布式存储系统里的【万金油】 里面的算法很好

一切皆对象；

数据写入采用主备复制模型

数据分布模型采用CRUSH算法



单机数据库

单个计算机节点上的数据库系统

事务在单机内执行 通过网络交互实现分布式事务



关系数据库

商业产品Oracle

开源产品 MySQL PostgreSQL

![image-20241127101722363](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127101722363.png)



![image-20241127101919309](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127101919309.png)

   

Page  *Redo Log*  临时文件

内存 与 磁盘之间的交互



![image-20241127102206620](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127102206620.png)



 非关系型数据库 没有准则 交互方式各不相同 

schema相对灵活 

SQL查询语言的统治地位很重视



![image-20241127102716222](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127102716222.png)





模糊搜索 



![image-20241127102924660](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127102924660.png)



Query： match 编程语言 好 难度  模糊匹配到小明的帖子 ES天然能够做 模糊搜索 还能自动算出关联程度 传统关系型无法做到这一点



分布式数据库（引入分布式架构）： 

容量 弹性 性价比 解决单机时代遇到的问题



解决容量问题：

![image-20241127103344147](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127103344147.png)



存储池 动态扩缩容



弹性问题：



![image-20241127103713521](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127103713521.png)

随着业务的变化而变化 1t如何搬到200g？

池化！



性价比问题

![image-20241127103908948](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127103908948.png)



硬件disk不需要怎么准备 什么不够用Storge Pool 分布式存储池来解决这个问题

磁盘池化 内存池化（降低成本）



单写vs多写     从磁盘弹性到内存弹性    分布式事务优化



##### 新技术演进



软件架构变更（依赖于操作系统内核） AI增强（智能存储格式转换） 新硬件革命（存储介质变更 计算单元变更 网络硬件变更）



![image-20241127104445468](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127104445468.png)

cpu中断 使性能降低 与之替换的是用轮询替换



![image-20241127104706603](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127104706603.png)

数据存储格式的转换 左边是二维表由 多个行与列 行存 列存 （优势 劣势）

ai决策-》行列混存（动态性强）

 

![image-20241127104909837](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127104909837.png)



![image-20241127105517074](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127105517074.png)



硬件反推软件变革



##### 课后作业



![image-20241127105647523](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127105647523.png)



材料引用

![image-20241127105742332](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127105742332.png)



#### 深入理解RDBMS（关系型数据库）

存储系统:

块存储 文件存储 对象存储 key-value存储



数据库系统：

大型关系数据库 非关系型数据库



抖音红包雨

从抖音账号扣除1个亿

![image-20241127110101367](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127110101367.png)





事务ACID

![image-20241127110324168](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127110324168.png)



红包雨与ACID 

![image-20241127110453611](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127110453611.png)

事务要同时成功与同时失败 



账户的钱不能为复数

![image-20241127110554045](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127110554045.png)

操作一定要是合法的



隔离性问题 ： 两个操作同时进行 有相互影响的关系

![image-20241127110756892](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127110756892.png)



刚开始抢了一个亿（成功） 但是服务器挂了 

![image-20241127111019483](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127111019483.png)



**高并发**

10亿人 同时抢红包 一定 数据要有处理高并发的能力 每秒处理事务的请求

**高可靠**

在关键时间和结点上 保证后台服务可靠



从 纸 到 磁盘文件



第一个数据库 网状数据库（W.Bachman）

​             Collage

English   computer  Maths

多对多

没有交叉 

结点 网络结构 父节点可以有多个子节点 





层次模型（IBM）

用树形结构描述实体 与 网状结构相似 但是并不交叉树状

每个子节点只有一个父节点

（1对多）



关系模型（IBM）（EFCodd博士）

![image-20241127113215926](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127113215926.png)



##### 关键技术



一条SQL的一生 ： 

解析SQL（语法解析器） 语法树AST  优化器 Plan Executor（执行器） 写入数据 写入日志



![image-20241127150742531](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127150742531.png)



分析



![image-20241127151110898](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127151110898.png)



为什么还要优化器？（Optimizer）  类比于 （高德地图路线优化） 快 慢 红绿灯 



基于规则的优化 

条件简化……

Scan优化



![image-20241127151708954](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127151708954.png)



基于代价优化：

时间是代价 最少时间到目的地

io cpu NET MEM也是代价



火山模型：

![image-20241127152543557](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127152543557.png)

向量化：

每次返回是一批数据 而不是一行数据

优点：

函数调用次数降低为1/N

CPU cache命中率更高

可以利用CPU提供SIMD机制 一次加法（可以操作多个数据）



编译执行：





##### 存储引擎-InnoDB



内存态（做一点内存缓存）

![image-20241127153304028](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127153304028.png)



存储引擎-Buffer Pool

instance0



instance1



HashMap管理



LRU 算法

保留最近最常使用的保存在内存 其他的 淘汰



内存放不下？ 放磁盘  从磁盘访问数据

![image-20241127153706131](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127153706131.png)



存储引擎-Page

 B+Tree索引 B树的扩展（二分查找树）

![image-20241127154133896](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127154133896.png)



##### 事务引擎：

原子性   与   Undo Log

同步失败或成功



如何将数据库回退到修改之前的状态？ 



Undo日志：

逻辑日志 进行事务回滚 保证原子性 



![image-20241127154501380](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127154501380.png)



isolation 与 锁  

如果两个操作同时发生 发生冲突怎么办

锁机制 

Share Lock 共享锁 读读 两个人都有共享锁

Exclusice Lock 写锁 读写 一个有写锁另一个不能有



读写   -》  MVCC数据的多版本

![image-20241127154917696](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241127154917696.png)



一致性：

对数据的修改 永久保存



方案一 事务提交前页面写盘

随机io 写放大 



方案二 WAL











##### 对象存储

申请Bucket -》 业务逻辑开发 -》 上线测试

TOS 



![image-20241129105018322](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129105018322.png)





对象存储： 简单的增删改查

![image-20241129105503283](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129105503283.png)



上传大文件       



![image-20241129105810331](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129105810331.png)



 ![image-20241129110625420](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129110625420.png)



接入层 ： 

高持久度 ： 用户视频不能丢 ！（硬性要求）



容量型



可扩展性： 容量/吞吐量可线性扩展

成本： 单位存储成本需要足够低

持久度： 如何在保证成本的情况下， 确保高持久度



QPS型



  可拓展性解法：Partition



分布式存储 = 分布式 ＋ 单机存储



分布式： 



持久度解法： Replication  （考虑到水灾 自然灾害 甚至是太阳黑子 不可靠的硬件 不可靠的软件）单结点存储一定是不行的 可能会丢失



多机架

多机房

多Region

 

![image-20241129112154811](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129112154811.png)



EC 冗余编码 可达到和多副本 一样的持久度



额外编码 

EC编码算法有哪些？

多机房EC如何实现 



高可用性：

SLA RPO RTO

服务一年只能5分钟不可用

事故5分钟之内解决！



一个集群拆分成多个集群 

![image-20241129113231607](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129113231607.png)



高可用之镜像灾备



容量治理 成本优化 大数据生态 稳定性提升



Go框架三件套详解

Web RPC GORM



GORM  Kitex Hertz（ORM   RPC框架   HTTP框架）



![image-20241129134635315](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129134635315.png)



  更新非零值  

什么是DSN



字段可以保持不一致

![image-20241129150829968](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129150829968.png)



Gorm链式调用



查询

![image-20241129151502394](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129151502394.png)



更新数据



![image-20241129151818730](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129151818730.png)



删除数据

![image-20241129152134491](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129152134491.png)



GORM 事务：



GORM文档

事务一致性

![image-20241129152535195](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129152535195.png)



固化一个连接应该使用tx



Gorm Hook





性能提高：

![image-20241129153137225](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129153137225.png)

关闭默认事务

缓存预编译语句

结构体复用

performance文档（性能提高文档）



![image-20241129153252041](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129153252041.png)

​    

悲观锁（redis） 乐观锁

读写分离 分库分表



看GORM文档提issue



Kitex对windows支持不完善 使用wsl2







![image-20241129154233850](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129154233850.png)







![image-20241129154654021](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129154654021.png)

   

kitex 服务注册与发现中心

常见的注册中心ETCD Nacos

内存的负载均衡

处理err

panic直接退出 还有 log.Fatal(err)





![image-20241129155145878](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129155145878.png)





![image-20241129155450290](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129155450290.png)



两个上下文？ 原信息 请求处理 两部分



![image-20241129155637239](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129155637239.png)



通配路由

![image-20241129160114059](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241129160114059.png)



