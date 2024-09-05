



# 总结所学

## 1.gin框架知识

## 第一天

 **1.gin框架打印 hello_world**

**2.学会创建路由 开放端口 根据相应路径访问相应函数**

**3.学会看gin框架中的各种方法例如： **

**原生http服务方式 router.run（）本质就是http.ListenAndsever()函数的封装 可以查看 router.Run的源码 **







代码如下：

~~~go
package main

import (
	"github.com/gin-gonic/gin"
)

//index的大小写 大写表示全局访问

func Index(context *gin.Context) {

	context.String(200, "hello world")
}
func main() {
	//创建默认路由
	router := gin.Default()
	//绑定路由规则和路由函数 访问 “/index”的路由的对应函数处理
	router.GET("/index", Index)
	//启动监听的端口
	router.Run(":8086")

	//原生http服务方式 router.run（）本质就是http.ListenAndsever()函数的封装 可以查看 router.Run的源码
	//http.ListenAndServe(":8086", router)
}
~~~

## 第二天

学习了几个响应 json  html  重定向

go语言的结构体的使用

**结构体的json标签格式 `json ：”user_name“**

**其中html响应  需要有一个template模板  里面设有html文件 插值语法 gin.H{username": "小赵"}} **

**注意html响应时候传入（结构体）的方法:**

~~~go
c.HTML(200, "index.html", gin.H{
		"User_name": user.Username,
		"Age":       user.Age,
	})

~~~

~~~html

<p>Age:{{.Age}}</p>
<p>Username:{{.User_name}}</p>

~~~



****

代码如下：

~~~go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func _string(c *gin.Context) {
	c.String(200, "你好")

}

//json响应是最重要的 其次还有xml响应 yaml响应

func _json(c *gin.Context) {
	//json响应结构体
	type UserInfo struct {
		Username string `json:"user_name"` //相当于 替换 Username
		Age      int    `json:"age"`       //同理
		Password string `json:"-"`         //这个json表示 - 表示不显示密码 忽略json  否则密码泄露很危险
	}

	user := UserInfo{Username: "小赵", Age: 20}

	//json响应map
	/*userMap := map[string]string{
		"user_name": "小赵",
		"age":       "20",
	}
	*/
	c.JSON(200, user)

}

// 响应html 得先让gin框架找到html模板在哪个位置
func _html(c *gin.Context) {

	type UserInfo struct {
		Username string `json:"user_name"` //相当于 替换 Username
		Age      int    `json:"age"`       //同理
		Password string `json:"-"`         //这个json表示 - 表示不显示密码 忽略json  否则密码泄露很危险
	}
	user := UserInfo{Username: "小赵", Age: 20}
	//gin.H作用加载后台传入的数据 和templates中的html文件相照应 插值语法 gin.H{username": "小赵"}}  下面这种写法是传入结构体
	c.HTML(200, "index.html", user)

}

// 重定向
func _redirect(c *gin.Context) {
	c.Redirect(301, "http://www.bilibili.com")
}

func main() {
	router := gin.Default()
	//记载模板目录下的所有模板文件
	router.LoadHTMLGlob("templates/*")

	//在golang中没有相对文件路径 只有相对项目路径
	//网页请求静态文件的前缀，第二个参数是目录
	router.StaticFS("/static", http.Dir("static/static"))

	//配置单个文件 使得 visible可访问 而 invisible不可访问 第一个参数：网页请求路径 第二个参数文件路径
	router.StaticFile("/image", "./static/image.png")

	router.GET("/", _string)
	router.GET("/json", _json)
	router.GET("/html", _html)
	router.GET("/bilibili", _redirect)
	router.Run(":89")
}
~~~

---

文件结构

* static
  * static
  * visible.txt

* image.png

* invisible.txt

  如下这段代码做到了显示该显示的文件  不显示不对外展示的文件

~~~go
//在golang中没有相对文件路径 只有相对项目路径
	//网页请求静态文件的前缀，第二个参数是目录
	router.StaticFS("/static", http.Dir("static/static"))
	
	//配置单个文件 使得 visible可访问 而 invisible不可访问 第一个参数：网页请求路径 第二个参数文件路径
	router.StaticFile("/image", "./static/image.png")

~~~

## 第三天

**1.学会使用postman 发送GET  POST 请求**

**2.postman可以接收  multipart/form-data;  application/x-www-form0urlencoded  等 Content-type**

1.查询参数

~~~go
// 查询参数 格式 url ？id= &user
func _query(c *gin.Context) {
	//user := c.Query("user")
	fmt.Println(c.GetQuery("user"))
	//fmt.Println(user)
	fmt.Println(c.Query("user"))
	fmt.Println(c.QueryArray("user")) //可以获取多个 user值
}
~~~

2.动态参数

~~~go
// 动态参数 访问路径 /param/xxx/xxx
func _param(c *gin.Context) {
	fmt.Println(c.Param("user_id"))
	fmt.Println(c.Param("book_id"))
}
~~~

3.表单参数

~~~go
// 表单参数 用postman发送请求 post请求 学会使用postman 在Body中填写key-value 而不是Params
// 可以接收 multipart/form-data; application/x-www-form0urlencoded 这两种 Content-type
func _form(c *gin.Context) {
	fmt.Println(c.PostForm("name"))
	fmt.Println(c.PostFormArray("name"))           //接受多个
	fmt.Println(c.DefaultPostForm("addr", "黑龙江省")) //如果用户没传就是默认值
	forms, err := c.MultipartForm()
	fmt.Println(forms, err)
}

~~~

4.封装一个解析json到结构体的函数

~~~go

// 封装了一个解析json到结构体上的函数
func bindJson(c *gin.Context, obj any) (err error) {

	body, _ := c.GetRawData()
	contentype := c.GetHeader("content-type")

	fmt.Println(contentype)
	//switch 的目的是挑选 类型为json的打印 其他类型的无法打印
	switch contentype {
	case "application/json":

		err := json.Unmarshal(body, &obj)
		if err != nil {
			fmt.Println(err.Error())
			return err
		}

	}
	return nil
}

// 原始参数 用form-data传 用x-www-form0urlencoded传 用json传都是表现出不同的格式   注释快捷键ctrl+/
func _ram(c *gin.Context) {
	type User struct {
		Name string `json:"name"` //必须得有json标签 否则无法解析 Unmarshal就是json序列化
		Age  int    `json:"age"`
	}
	var user User
	err := bindJson(c, &user)//使用了方法
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(user)

~~~



~~~go
body, _ := c.GetRawData()
	contentype := c.GetHeader("content-type")
	
	fmt.Println(contentype)
	//switch 的目的是挑选 类型为json的打印 其他类型的无法打印
	switch contentype {
	case "application/json":
		type User struct {
			Name string `json:"name"` //必须得有json标签 否则无法解析 Unmarshal就是json序列化
			Age  int    `json:"age"`
		}
		var user User
		err := json.Unmarshal(body, &user)
		if err != nil {
			fmt.Println(err.Error())
		}
		fmt.Println(user)
	}
~~~

## 第四天

1.四大请求方法

**GET从服务器获取出资源**

**POST：从服务器新建资源**

**PUT：从服务器更新资源**

**PATCH：从服务器更新资源**

**DELETE从服务器更新资源**

~~~
//以文章资源为例 增删改查四个接口

//GET articles         文章列表
//GET /articles/：id   文章详细
//POST  /articles      添加文章
//PUT /articles/:id    修改某一篇文章
//DELETE /articles/:id 删除某一篇文章 


~~~

思路两个结构体代码：

~~~go

type ArticleModel struct {
	Title   string `json:"title"`
	Content string `json:"content"`
}

type Response struct {
	Code int    `json:"code"`
	Data any    `json:"data"`
	Msg  string `json:"msg"`
}

~~~



函数的目的是在Gin框架中提供一个通用的方式来解析请求数据为JSON

~~~go
/ 封装了一个解析json到结构体上的函数
func _bindJson(c *gin.Context, obj any) (err error) {

	body, _ := c.GetRawData()
	contentype := c.GetHeader("content-type")

	fmt.Println(contentype)
	//switch 的目的是挑选 类型为json的打印 其他类型的无法打印
	switch contentype {
	case "application/json":
		//json.Unmarshal(body, &obj)
		err := json.Unmarshal(body, &obj)
		if err != nil {
			fmt.Println(err.Error())
			return err
		}

	}
	return nil
}
~~~

---

文章列表页面

~~~go

// 文章列表页面
func _getList(c *gin.Context) {

	articles := []ArticleModel{
		{Title: "GO语言入门", Content: "这篇文章内容《go》"},
		{Title: "python语言入门", Content: "这篇文章内容《python》"},
		{Title: "css语言入门", Content: "这篇文章内容《css》"},
	}
	c.JSON(200, Response{Code: 200, Data: articles, Msg: "成功"})
}

~~~

---

文章详细页面 通常需要数据库

~~~go
// 文章详情页 但是目前省去了数据库查找操作
func _getDetail(c *gin.Context) {
	articles := []ArticleModel{
		{Title: "GO语言入门", Content: "这篇文章内容《go》"},
	}

	//获取param中的id
	fmt.Println(c.Param("id"))
	c.JSON(200, Response{Code: 200, Data: articles, Msg: "成功"})
}

~~~

---

创建文章

```go
func _create(c *gin.Context) {
    //接收前端传进来的JSON数据
    var article ArticleModel

    err := _bindJson(c, &article)
    if err != nil {
       fmt.Println(err.Error())
       return
    }
    c.JSON(0, Response{Code: 200, Data: article, Msg: "添加成功"})
```

创建成功标志

~~~
/*成功标志
	{
	    "code": 200,
	    "data": {
	        "title": "要添加一个文章",
	        "content": "我现在用postman中的POST请求添加文章  在BOody中的JSON格式"
	    },
	    "msg": "添加成功"
	}
	*/
}
~~~

---

编辑文档学会**运用postman中的Boby 中raw填入JSON格式的数据**

~~~go
func _update(c *gin.Context) {
	fmt.Println(c.Param("id"))
	var article ArticleModel

	err := _bindJson(c, &article)
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	c.JSON(200, Response{Code: 200, Data: article, Msg: "修改成功"})
}
~~~

---

删除文档

```go
// 删除文章根据id把文章删掉
func _delete(c *gin.Context) {

    fmt.Println(c.Param("id"))
    c.JSON(200, Response{Code: 200, Data: map[string]string{}, Msg: "删除成功"})
}
```

路由设置

```go
func main() {
    router := gin.Default()
    router.GET("/articles", _getList)       //文章列表
    router.GET("/articles/:id", _getDetail) //文章详细
    router.POST("/articles", _create)       //添加文章
    router.PUT("/articles/:id", _update)    //编辑文章
    router.DELETE("/articles/:id", _delete) //删除文章
    router.Run(":888")
}
```

## 第五天

**请求头相关**

**请求头参数获取**

~~~go
//请求头的各种获取方式
	router.GET("/", func(c *gin.Context) {
		//用这种方式获得一个请求头
		fmt.Println(c.GetHeader("User-Agent"))
		fmt.Println(c.GetHeader("User-agent"))
		fmt.Println(c.GetHeader("User-Agent"))
		//请求整个请求头 Header本身是一个普通的 map[string][]string
		fmt.Println(c.Request.Header)
		c.JSON(200, gin.H{"msg": "成功"})
	})
~~~





**响应头相关**

怎么设置响应头













**bind绑定参数**

gin中的bind可以很方便的将前端传递来的数据与`结构体`进行`参数绑定`，以及参数校验

**参数绑定**

在使用这个功能的时候，需要给结构体加上Tag `json` `form` `url` `xml` `yaml`

**方法**

**1.MustBind**

不用 校验失败会改变状态码

**2.ShouldBindJOSN**

会根据请求头中的content-type去自动绑定

form-data中的参数也用这个 tag用form

默认的tag就是form

绑定form-data x-www-urlemcode



~~~go

type UserInfo struct {
	Name string  form:"name" `
	Age  int    ` form:"age" `
	Sex  string `form:"sex" `
}



	router.POST("/form", func(c *gin.Context) {
		var userinfo UserInfo
		err := c.ShouldBind(&userinfo)
		if err != nil {
			fmt.Println(err)
			c.JSON(200, gin.H{"msg": "你错了"})
		}
		c.JSON(200, userinfo)
	})

	router.Run(":804")



~~~

---

**3.ShouldbindQuery**

~~~go
//绑定查询参数
	//http://127.0.0.1:804/query?name=赵忠鹤&age=男&sex=男
	//tag 对应form
	router.POST("/query", func(c *gin.Context) {
		var userinfo UserInfo
		err := c.ShouldBindQuery(&userinfo) //以及ShouldBind
		if err != nil {
			fmt.Println(err)
			c.JSON(200, gin.H{"msg": "你错了"})
			return
		}
		c.JSON(200, userinfo)
	})
~~~

4.SHouldBinduri

~~~go
	//绑定动态参数  uri/:name/:age/:sex
	router.POST("uri/:name/:age/:sex", func(c *gin.Context) {
		var userinfo UserInfo
		err := c.ShouldBindUri(&userinfo)
		if err != nil {
			fmt.Println(err)
			c.JSON(200, gin.H{"msg": "错误"})
			return
		}
		c.JSON(200, userinfo)

	})
~~~

## 第六天

**常用校验器**

~~~
//不能为空，并且不能没有这个字段
required:必填字段，如:binding:"required"
min最小长度，如:binding:"min=5 "
max是大长度，如:binding:"max=10" 
Len 长度，如:binding:"Len=6" 
ea等于，如:binding:"eq=3 "
ne不等于，如:binding:"ne=12" 
gt大于，如:binding:"gt=10"
gte大于等于，如:binding:*gte=10" 
lt小于，如:binding:lt=18 
Lte小于等于，如:binding:"Lte=18" 
eafietd等于其他字段的值，如:PassWord string 'binding:"eafietd=ConfirmPassword"
nefied 不等于其他字段的zhi


~~~

**内置校验器**



**1.校举只能是red 或green** 
oneof=red green 
**2.字符串**

contains=fengfeng//包含fengfeng的字符串
excLudes I/不包含
startswith//字符串前线
endwith/字符串后缀
**3.数组**
dive  // dive后面的验证是针对数组中的海一个元素
**4.网络验证**
ip 
ipv4 
ipv6 
uri 
url
// uri 在于ICIdentifier)是统一资源标示符，可以唯一标识-个资源。 
// url在于Lgcater，是统一资源定位符，提供找到该资源的确切路径

**5.日期验证**

datetime=2006-01-02

---

**自定义错误信息**



