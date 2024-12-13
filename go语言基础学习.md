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





### Go进阶

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



### 中间件学习
