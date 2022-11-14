# gin基础


# gin基础

## HelloWord

```go
func main() {
	// 1.创建路由 默认使用了2个中间件Logger(), Recovery()
	r := gin.Default()
	// 2.绑定路由规则，执行的函数
	// gin.Context，封装了request和response
	r.GET("/", func(c *gin.Context) {
		c.String(http.StatusOK, "hello World!")
	})
	// 监听端口，默认在8080
	// Run("里面不指定端口号默认为8080")
	r.Run(":8000")
}

```

## 请求接收

### 路由参数

> `:` 匹配一个
>
> `* ` 匹配0个或多个
>
> 127.0.0.1:8888/user/1/1234 		 	 匹配
>
> 127.0.0.1:8888/user/1/1234/1234 	匹配
>
> 127.0.0.1:8888/user/1						  匹配

```go
	//api 参数 Param 方法
	r.GET("/user/:name/*action", func(c *gin.Context) {
		name := c.Param("name")
		action := c.Param("action")
		//截取/
		action = strings.Trim(action, "/")
		c.String(http.StatusOK, name+" is "+action)
	})
```

### URL参数

> DefaultQuery : 获取不到取默认值
>
> Query : 获取参数
>
> 127.0.0.1:8888/user?action=1234

```go
	//url参数 DefaultQuery Query
	r.GET("/user", func(c *gin.Context) {
		name := c.DefaultQuery("name", "哈哈哈")
		action := c.Query("action")
		c.String(http.StatusOK, name+" is "+action)
	})
```

### 表单参数

```go
	//表单参数 表单参数可以通过PostForm()方法获取，该方法默认解析的是x-www-form-urlencoded或from-data格式的参数
	r.POST("/form", func(c *gin.Context) {
		types := c.DefaultPostForm("type", "post")
		username := c.PostForm("username")
		password := c.PostForm("userpassword")
		c.String(http.StatusOK, fmt.Sprintf("username:%s,password:%s,type:%s", username, password, types))
	})
```

### JSON参数

```go
	//json参数
	// 定义接收数据的结构体
	type Login struct {
		// binding:"required"修饰的字段，若接收为空值，则报错，是必须字段
		// form：表单
		// json：json
		// uri：url参数
		User     string `form:"username" json:"user" uri:"user" xml:"user" binding:"required"`
		Password string `form:"password" json:"password" uri:"password" xml:"password" binding:"required"`
	}

	//数据绑定
	r.POST("/loginJSON", func(c *gin.Context) {
		// 声明接收的变量
		var json Login
		// c.ShouldBindJSON 将request的body中的数据，自动按照json格式解析到结构体
		// c.ShouldBindUri 解析url参数
		// c.Bind Bind()默认解析并绑定form格式 根据请求头中content-type自动推断
		if err := c.Bind(&json); err != nil {
			// 返回错误信息
			// gin.H封装了生成json数据的工具
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		// 判断用户名密码是否正确
		if json.User != "root" || json.Password != "admin" {
			c.JSON(http.StatusBadRequest, gin.H{"status": "304"})
			return
		}
		c.JSON(http.StatusOK, gin.H{"status": "200"})
	})
```

### 文件上传

```go
//文件上传大小限制 单位k
	r.MaxMultipartMemory = 8 << 20
	//单文件上传
	r.POST("/upload", func(c *gin.Context) {
		file, err := c.FormFile("file")
		if err != nil {
			c.String(500, "上传图片出错")
		}
		//文件存储
		c.SaveUploadedFile(file, "/Users/mr_j/"+file.Filename)
		c.String(http.StatusOK, file.Filename)
	})

	//多文件
	r.POST("/uploadBatch", func(c *gin.Context) {
		form, err := c.MultipartForm()
		if err != nil {
			c.String(http.StatusBadRequest, fmt.Sprintf("get err %s", err.Error()))
		}
		// 获取所有图片
		files := form.File["files"]
		// 遍历所有图片
		for _, file := range files {
			// 逐个存
			if err := c.SaveUploadedFile(file, "/Users/mr_j/"+file.Filename); err != nil {
				c.String(http.StatusBadRequest, fmt.Sprintf("upload err %s", err.Error()))
				return
			}
		}
		c.String(200, fmt.Sprintf("upload ok %d files", len(files)))
	})
```

## 响应

### JSON

```go
	r.GET("/someJSON", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "someJSON", "status": 200})
	})
```

### 结构体

```go
	r.GET("/someStruct", func(c *gin.Context) {
		var msg struct {
			Name    string
			Message string
			Number  int
		}
		//是否可以添加构造方法

		msg.Name = "root"
		msg.Message = "message"
		msg.Number = 123
		c.JSON(200, msg)
	})
```

### 其他

```go
	// 3.XML
	r.GET("/someXML", func(c *gin.Context) {
		c.XML(200, gin.H{"message": "abc"})
	})
	// 4.YAML响应
	r.GET("/someYAML", func(c *gin.Context) {
		c.YAML(200, gin.H{"name": "zhangsan"})
	})
	// 5.protobuf格式,谷歌开发的高效存储读取的工具
	// 数组？切片？如果自己构建一个传输格式，应该是什么格式？
	r.GET("/someProtoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		// 定义数据
		label := "label"
		// 传protobuf格式数据
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		c.ProtoBuf(200, data)
	})
```

## 路由拆分

### 同文件拆分

```go
func main() {
	r := gin.Default()
	//路由分组
	// 路由组1 ，处理GET请求
	v1 := r.Group("/v1")
	// {} 是书写规范
	{
		v1.GET("/login", login)
		v1.GET("/submit", submit)
	}
	v2 := r.Group("/v2")
	{
		v2.POST("/login", login)
		v2.POST("/submit", submit)
	}
	// 监听端口，默认在8080
	// Run("里面不指定端口号默认为8080")
	r.Run(":8000")
}

func login(c *gin.Context) {
	name := c.DefaultQuery("name", "jack")
	c.String(200, fmt.Sprintf("hello %s\n", name))
}

func submit(c *gin.Context) {
	name := c.DefaultQuery("name", "lily")
	c.String(200, fmt.Sprintf("hello %s\n", name))
}
```

### 单独拆分路由文件

1. `./main.go`

   ```go
   func main() {
   	r := setupRouter()
   	if err := r.Run(":8000"); err != nil {
   		fmt.Printf("startup service failed, err:%v\n", err)
   	}
   }
   ```

2. `./routers.go`

   ```go
   func helloHandler(c *gin.Context) {
   	c.JSON(http.StatusOK, gin.H{
   		"message": "Hello www.topgoer.com!",
   	})
   }

   func setupRouter() *gin.Engine {
   	r := gin.Default()
   	r.GET("/topgoer", helloHandler)
   	return r
   }
   ```

### 按照包拆分路由文件

1. `./main.go`

   ```go
   package main

   // 按照包拆分路由文件
   func main() {
   	r := routers.SetupRouter()
   	if err := r.Run(":8000"); err != nil {
   		fmt.Printf("startup service failed, err:%v\n", err)
   	}
   }
   ```

2. `./routers/routers.go`

   ```go
   package routers

   func helloHandler(c *gin.Context) {
   	c.JSON(http.StatusOK, gin.H{
   		"message": "Hello www.topgoer.com!",
   	})
   }

   func SetupRouter() *gin.Engine {
   	r := gin.Default()
   	r.GET("/topgoer", helloHandler)
   	return r
   }

   ```

### 路由拆分成多个文件

1. `./main.go`

   ```go
   package main

   // 路由拆分成多个文件
   func main() {
   	r := gin.Default()
   	//加载路由
   	routers.LoadUser(r)
   	routers.LoadBlog(r)
   	if err := r.Run(":8000"); err != nil {
   		fmt.Printf("startup service failed, err:%v\n", err)
   	}
   }
   ```

2. `./routers/user.go`

   ```go
   package routers

   func LoadUser(e *gin.Engine) {
   	loadUser := e.Group("/user")
   	{
   		loadUser.GET("/:id", getById)
   		loadUser.DELETE("/:id", deleteById)
   	}
   }

   func deleteById(c *gin.Context) {
   	c.String(http.StatusOK, "hello World!")
   }

   func getById(c *gin.Context) {
   	c.String(http.StatusOK, "hello World!")
   }
   ```

3. `./routers/blog.go`

   ```go
   package routers

   func LoadBlog(e *gin.Engine) {
   	loadBlog := e.Group("/blog")
   	{
   		loadBlog.GET("/:id", getBlogById)
   		loadBlog.DELETE("/:id", deleteBlogById)
   	}
   }

   func getBlogById(c *gin.Context) {
   	c.String(http.StatusOK, "hello World!")
   }

   func deleteBlogById(c *gin.Context) {
   	c.String(http.StatusOK, "hello World!")
   }

   ```

## 中间件

```go

// MiddleWare 定义
func MiddleWare() gin.HandlerFunc {
	return func(c *gin.Context) {

		fmt.Println("中间件开始执行了")

		req := c.Query("request")
		// 设置变量到Context的key中，可以通过Get()取
		c.Set("request", "成功")
		if req == "1" {
			//执行处理
			c.Next()
		} else {
			c.JSON(500, gin.H{"error": req})
			//不执行处理 但是这个中间件中代码会执行完
			c.Abort()
		}
		fmt.Println("返回时执行")
	}
}

func main() {
	// 1.创建路由
	// 默认使用了2个中间件Logger(), Recovery()
	r := gin.Default()
	// 注册中间件
	r.Use(MiddleWare())
	// {}为了代码规范
	{
		r.GET("/ce", func(c *gin.Context) {
			// 取值
			req, _ := c.Get("request")
			fmt.Println("request:", req)
			// 页面接收
			c.JSON(200, gin.H{"request": req})
		})

	}
	r.Run()
}
```


