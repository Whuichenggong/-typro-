# go项目学习

## gin-blog-admin

**1.首先遇到的问题是docker**

docker镜像源的失效 导致拉取资源失败

~~~
解决方法
去chorm搜索 docker加速源 更换加速源就可以正常拉取资源和文件

docker运行成功
~~~

## 一.需要学习的知识点

### 1.搞清楚项目目录结构和功能

### 2.dockerfile文件指令

### 3.swagger ui

### 4.数据库 mysql与redis的连接

### 5.中间件的设计与学习



## 二.jsconfig.json文件学习

~~~json
{
  "compilerOptions": {
    "baseUrl": ".",
    "target": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "paths": {
      "@/*": ["./src/*"],
      "~/*": ["./*"]
    }
  },
  "exclude": ["dist", "node_modules"]
}

~~~

在 `"paths"` 配置中：

- `"@/*": ["./src/*"]`：
  - 这是设置了一个路径别名。当在代码中使用以 `"@/"` 开头的路径时，TypeScript 编译器会将其解析为 `./src/` 加上后面相对应的路径部分。
  - 例如，在代码中如果有 `import something from '@/component'`，那么实际上会被解析为从 `./src/component` 导入。这样可以方便地引用项目中特定目录下的模块，尤其是当项目结构比较复杂或者需要统一的路径引用方式时。
- `"~/*": ["./*"]`：
  - 类似地，这里设置了另一个别名。当使用以 `"~/"` 开头的路径时，会被解析为当前目录（`./`）加上后面相对应的路径部分。
  - 比如，`import something from '~/utils'` 可能会被解析为从当前目录下的 `./utils` 导入模块。

这种配置方式可以提高代码的可读性和可维护性，避免使用相对较长或复杂的相对路径进行模块导入。



**一、“exclude” 的总体作用**

“exclude” 选项用于指定在编译、检查或其他处理过程中应该被排除的文件或目录。

**二、具体排除的内容及原因**

1. 排除 “dist” 目录：
   - “dist” 目录通常是存放编译后的输出文件的地方。在开发过程中，一般不需要对已经编译生成的文件进行再次处理，比如类型检查或编译。排除这个目录可以提高处理速度，避免不必要的资源占用。
   - 例如，在 TypeScript 项目中，编译后的 JavaScript 文件通常会存放在 “dist” 目录下。如果不排除这个目录，编译器可能会浪费时间去检查已经编译好的代码，而不是专注于原始的源代码。
2. 排除 “node_modules” 目录：
   - “node_modules” 目录包含了项目所依赖的第三方库和模块。这些库通常是已经编译好的代码，并且不需要进行类型检查或编译。排除这个目录可以大大加快处理速度，特别是在大型项目中，“node_modules” 目录可能非常庞大。
   - 此外，很多第三方库可能没有类型定义文件，或者其类型定义文件可能存在一些问题。如果不排除 “node_modules” 目录，可能会导致编译器出现错误或警告，影响开发效率。

总之，通过设置 “exclude” 选项来排除 “dist” 和 “node_modules” 目录，可以提高开发工具的性能，减少不必要的处理和错误，让开发过程更加高效和顺畅。



## 三.逐步分析Dockerfile文件

### 1.根目录的 dockerfile文件



~~~dockerfile
# 阶段一: 打包前后台静态资源
FROM node:18-alpine3.19 AS BUILD
WORKDIR /app/front
COPY gin-blog-front/package*.json .
RUN npm config set registry https://registry.npmmirror.com \
    && npm install -g pnpm \ 
    && pnpm install
COPY gin-blog-front .
RUN pnpm build

WORKDIR /app/admin
COPY gin-blog-admin .
RUN pnpm install && pnpm build

~~~

**第八行代码**

这个指令的目的是将特定的项目文件整合到 Docker 镜像中，以确保容器在运行时能够访问和使用这些文件。



**一、基础镜像和阶段命名**

- `FROM node:18-alpine3.19 AS BUILD`：使用 Node.js 18 的 Alpine Linux 版本作为基础镜像，并将这个阶段命名为 “BUILD”。Alpine Linux 是一个轻量级的 Linux 发行版，能使镜像体积较小，启动速度较快。

**二、前台构建步骤**

1. `WORKDIR /app/front`：设置工作目录为`/app/front`。

2. `COPY gin-blog-front/package*.json.`：将名为 “gin-blog-front” 的项目中的`package*.json`文件复制到当前工作目录。这通常是为了在后续步骤中安装项目依赖。

3. ```
   RUN npm config set registry https://registry.npmmirror.com && npm install -g pnpm && pnpm install
   ```

   - 首先设置 npm 的镜像源为`https://registry.npmmirror.com`，通常是为了加快包的下载速度和提高稳定性。
   - 然后全局安装`pnpm`（可能是一种包管理工具）。
   - 最后使用`pnpm install`安装当前项目的依赖。

4. `COPY gin-blog-front.`：将整个 “gin-blog-front” 项目复制到当前工作目录。

5. `RUN pnpm build`：使用`pnpm`执行项目的构建命令，可能会编译、打包前端资源等。



~~~dockerfile
# 阶段二: 将静态资源部署到 Nginx
FROM nginx:1.24.0-alpine

# 从第一个阶段拷贝构建好的静态资源到容器
COPY --from=BUILD /app/front/dist /usr/share/nginx/html
COPY --from=BUILD /app/admin/dist /usr/share/nginx/html/admin

# 将 Nginx 配置文件模板拷到容器中, 并执行脚本填充环境变量
COPY deploy/build/web/default.conf.template /etc/nginx/conf.d/default.conf.template
COPY deploy/build/web/default.conf.ssl.template /etc/nginx/conf.d/default.conf.ssl.template
COPY deploy/build/web/run.sh /docker-entrypoint.sh
RUN chmod a+x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

CMD [ "nginx", "-g", "daemon off;" ]

EXPOSE 80
EXPOSE 443

~~~

**一、基础镜像选择**

- `FROM nginx:1.24.0-alpine`：使用 Nginx 1.24.0 的 Alpine 版本作为基础镜像。Alpine Linux 是轻量级的，使得最终的镜像体积较小。

**二、静态资源复制**

- `COPY --from=BUILD /app/front/dist /usr/share/nginx/html`：从第一阶段构建的镜像（名为 “BUILD”）中，将前台项目构建后的静态资源（位于 `/app/front/dist`）复制到 Nginx 镜像中的 `/usr/share/nginx/html` 目录。这个目录通常是 Nginx 服务对外提供静态文件的根目录，这样就可以通过 Nginx 来提供前台静态资源的访问服务。
- `COPY --from=BUILD /app/admin/dist /usr/share/nginx/html/admin`：类似地，将后台项目构建后的静态资源（位于 `/app/admin/dist`）复制到 `/usr/share/nginx/html/admin`，为后台静态资源提供特定的访问路径。

**三、配置文件处理**

- `COPY deploy/build/web/default.conf.template /etc/nginx/conf.d/default.conf.template`：将部署目录下的 Nginx 配置文件模板（可能包含一些可替换的变量或占位符）复制到 Nginx 镜像中的 `/etc/nginx/conf.d/default.conf.template`。
- `COPY deploy/build/web/default.conf.ssl.template /etc/nginx/conf.d/default.conf.ssl.template`：可能是与 SSL 配置相关的另一个模板文件，同样复制到配置目录。
- `COPY deploy/build/web/run.sh /docker-entrypoint.sh`：将一个脚本文件（可能用于在容器启动时进行一些初始化操作，如填充环境变量、生成最终的配置文件等）复制到容器中并命名为 `/docker-entrypoint.sh`。
- `RUN chmod a+x /docker-entrypoint.sh`：为脚本文件添加可执行权限。

**四、容器启动命令**

- `ENTRYPOINT ["/docker-entrypoint.sh"]`：指定容器启动时执行的入口点脚本为 `/docker-entrypoint.sh`，这个脚本可能会根据模板生成实际的 Nginx 配置文件，并进行一些其他的初始化操作。
- `CMD [ "nginx", "-g", "daemon off;" ]`：指定容器启动后的默认命令为启动 Nginx 服务，并设置为非守护进程模式（`daemon off`），这样容器在前台运行 Nginx，方便查看日志和进行调试。

**五、端口暴露**

- `EXPOSE 80`：声明容器将在 80 端口提供服务，通常是 HTTP 服务端口。
- `EXPOSE 443`：声明容器将在 443 端口提供服务，通常是 HTTPS 服务端口。

总体来说，这段代码的作用是构建一个基于 Nginx 的容器镜像，将前后台项目构建后的静态资源复制到容器中，并通过配置文件模板和入口点脚本进行 Nginx 的配置，最终暴露 HTTP 和 HTTPS 服务端口，以便对外提供前后台静态资源的访问服务。



### 2.如何查找路径

~~~
# 从第一个阶段拷贝构建好的静态资源到容器
COPY --from=BUILD /app/front/dist /usr/share/nginx/html
COPY --from=BUILD /app/admin/dist /usr/share/nginx/html/admin
~~~

用于将构建阶段生成的前端应用程序的打包文件（通常是一个名为“dist”的文件夹）复制到 Nginx 服务器的网页根目录（/usr/share/nginx/html）

**一、在 Docker 环境中查找**

使`docker ps` 命令找到正在运行的包含 Nginx 的容器 ID 或名称。

使用 `docker exec -it <容器 ID 或名称> bash` 进入容器的命令行界面。

在容器内使用 `ls -l /usr/share/nginx/html` 命令查看该路径下的内容，确认是否有从前端构建目录复制过来的文件。

###  3.查看 Nginx 配置文件

1. Nginx 的配置文件中可能会指定网页根目录的路径。通常在 

   ```
   /etc/nginx/nginx.conf
   ```

   ```
   /etc/nginx/sites-enabled/
   ```

   - 查找类似 `root /usr/share/nginx/html;` 的配置行，确认该路径是否正确。

     docker ps 展开容器 如果想查看某个文件 就要进入容器并查看文件

     

     **在dockerdesktop中可以启动项目容器 在容器内我们又可以看到 很多镜像文件 在每个分开的容器中我们又可以查看我们在Dockerfile中的指令**

     要进入容器，可以使用以下命令：`**docker exec -it <容器 ID 或名称> /bin/sh**
     
     
     
     ## 配置文件学习
     
     在 deply/build/mysql中查看配置文件
     
     
     
     首先在dockerfile文件中
     
     ~~~dockerfile
     FROM mysql:8.0
     
     # 定义工作目录
     ENV WORK_PATH /usr/local/work
     # 定义被容器自动执行的目录
     ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d
     # 定义要执行的 shell 文件
     ENV RUN_SHELL run.sh
     
     COPY ./mysql.cnf /etc/mysql/conf.d/
     # 把数据库初始化数据的文件复制到工作目录
     COPY ./gvb.sql ${WORK_PATH}/
     # 把执行初始化的脚本放到 /docker-entrypoint-initdb.d 目录下
     COPY ./${RUN_SHELL} ${AUTO_RUN_DIR}/
     
     # 给执行文件添加可执行权限
     RUN chmod a+x ${AUTO_RUN_DIR}/${RUN_SHELL}
     ~~~
     
     1.工作目录在docker中可以进入docker中查看 路径中的文件
     
     2.根据dockerfile中的文件 可以一步步找到该看什么文件
     
     

## 四.三种数据库的区别

=====

- 🌟 答案:

**一、MySQL、SQLite 和 Redis 的区别**

1. **MySQL**：
   - 是一种功能强大的关系型数据库管理系统，适用于存储大量结构化数据，支持复杂的查询和事务处理。
   - 通常用于需要长期存储、数据完整性要求高、多用户并发访问的场景。
   - 可以处理大规模的数据，具有较高的性能和可扩展性。
2. **SQLite**：
   - 轻量级的嵌入式数据库，无需服务器进程，直接以文件形式存储数据。
   - 适合小型应用、移动应用或临时数据存储。在一些场景下，可能用于存储配置信息、缓存数据或者作为测试环境中的简单数据库。
   - 对于一些简单的应用场景，SQLite 可以快速部署，并且不需要复杂的安装和配置过程。
3. **Redis**：
   - 是一种内存数据库，但也可以将数据持久化到磁盘。它主要用于缓存、消息队列、分布式锁等场景。
   - 具有极高的读写性能，非常适合存储频繁访问的数据，以提高系统的响应速度。
   - 支持多种数据结构，如字符串、列表、哈希表、集合等，方便不同类型数据的存储和操作。

**二、一个项目使用三个数据库的原因**

1. 性能优化：
   - MySQL 用于存储长期的、大量的结构化数据。Redis 可以作为缓存，存储频繁访问的数据，减少对 MySQL 的访问压力，提高系统的响应速度。
   - SQLite 可能在某些特定场景下使用，比如在开发过程中进行快速测试，或者存储一些临时的、不太重要的数据。
2. 功能需求：
   - 不同的数据库具有不同的功能特点。Redis 的数据结构丰富，可以满足一些特殊的业务需求，如队列操作、排行榜等。而 MySQL 和 SQLite 则更适合传统的关系型数据存储。
3. 灵活性：
   - 根据不同的业务需求和场景，选择合适的数据库可以提高系统的灵活性和可扩展性。例如，在某些情况下，可能需要快速切换数据库或者使用不同的数据库来满足不同的业务模块需求。

总之，一个项目使用多个数据库是为了充分利用不同数据库的优势，以满足项目的性能、功能和灵活性需求。



## 五..sh文件作用

.sh 文件是一种 shell 脚本文件，其主要作用有以下几个方面：

**一、自动化任务**

1. 可以用来自动化执行一系列命令，比如重复的系统管理任务、软件安装过程、数据处理流程等。例如，可以编写一个.sh 文件来自动备份数据库、清理临时文件、编译和部署软件等。
2. 可以设置定时任务，让脚本在特定的时间自动运行，提高工作效率。

**二、系统管理**

1. 用于管理操作系统的各种配置和服务。例如，可以编写一个.sh 文件来修改系统环境变量、启动或停止服务、配置网络设置等。
2. 方便进行系统维护和故障排除。例如，可以编写一个脚本，在系统出现问题时自动收集诊断信息并发送给管理员。

**三、软件开发**

1. 在软件开发过程中，可以使用.sh 文件来构建、测试和部署软件。例如，可以编写一个构建脚本，自动编译代码、运行测试用例、打包软件等。
2. 方便开发人员在不同的开发环境中快速设置和配置项目。例如，可以编写一个脚本，自动安装项目所需的依赖库、配置开发环境变量等。

**四、跨平台性**

1. 由于大多数操作系统都支持某种形式的 shell 脚本，所以.sh 文件具有一定的跨平台性。虽然不同的操作系统可能使用不同的 shell（如 Linux 常用 Bash，Windows 可以使用 PowerShell 或 Git Bash 等），但基本的语法和功能相似，可以在不同平台上进行适当修改后使用。

总之，.sh 文件是一种非常强大的工具，可以帮助用户提高工作效率、简化系统管理和软件开发过程。



~~~sh
cd create-superadmin

# 创建一个用户名 superadmin, 密码 superadmin 的超级管理员
# 超级管理员可以访问所有页面, 所有资源
go run main.go -username superadmin -password superadmin
~~~

这段.sh文件 先是移动到了 一个文件夹中 并且执行了 go run main .go 创建了一个用户



## 六.swag指令



“swag init -g./cmd/main.go” 是用于生成 Go 语言项目的 API 文档的命令。

**一、关于 Swag**

Swag 是一个用于 Go 语言的工具，它可以根据代码中的注释自动生成 API 文档，并提供了一个方便的方式来组织和展示 API 信息。

**二、命令解释**

1. “swag init”：这是启动 Swag 生成文档的主要命令。
2. “-g./cmd/main.go”：这个参数指定了项目的入口文件，Swag 会从这个文件开始分析项目的代码结构和注释，以生成 API 文档。

**三、生成的内容**

Swag 会生成以下内容：

1. docs 目录：包含生成的 API 文档，通常以 HTML、Markdown 等格式呈现。
2. swagger.yaml 或 swagger.json 文件：这是 OpenAPI 规范的描述文件，包含了 API 的详细信息，如路径、方法、参数、响应等。

通过使用 Swag，可以提高项目的可维护性和可读性，方便其他开发人员了解和使用项目的 API。同时，也可以方便地与其他工具集成，如 API 测试工具、文档生成工具等。



## 七.观察main.go

### 1.控制台输出很重要

**示例**

generate-data文件中的main.go输出

~~~shell
go run main.go
2024/09/21 19:00:39 配置文件内容加载成功:  ../../config.yml
2024/09/21 19:00:39 数据库连接成功 mysql root:123456@tcp(127.0.0.1:3306)/gvb?charset=utf8mb4&parseTime=True&loc=Local
2024/09/21 19:00:39 数据库自动迁移成功
2024/09/21 19:00:39 INFO -----初始化博客配置 start-----
2024/09/21 19:00:39 INFO -----初始化博客配置 end-----
2024/09/21 19:00:39 INFO -----初始化博客页面 start-----
2024/09/21 19:00:39 INFO -----初始化博客页面 end-----
2024/09/21 19:00:39 INFO -----初始化默认角色用户 start-----
2024/09/21 19:00:39 INFO -----初始化默认角色用户 end-----
2024/09/21 19:00:39 INFO -----初始化接口资源 start-----
2024/09/21 19:00:40 INFO -----初始化接口资源 end-----
2024/09/21 19:00:40 INFO -----初始化菜单 start-----
2024/09/21 19:00:40 INFO -----初始化菜单 end-----
~~~

逐渐跟进



**通过观察main.go的日志输出来进一步跟进项目 观察创建的结构体 结构体之间的嵌套 以及结构体调用的方**



### 2.包的导入

~~~go

import (
	"flag"
	ginblog "gin-blog/internal"
	g "gin-blog/internal/global"
	"gin-blog/internal/middleware"
	"log"
	"strings"

	"github.com/gin-gonic/gin"
)
~~~

*包名ginblog  路径"gin-blog/internal"*

*包名g   路径"gin-blog/internal/global" *  这两个导入很有意思

**ginblog  g 代表两个包** 

~~~go
// 根据命令行参数读取配置文件, 其他变量的初始化依赖于配置文件对象
	conf := g.ReadConfig(*configPath)

	_ = ginblog.InitLogger(conf)
	db := ginblog.InitDatabase(conf)
	rdb := ginblog.InitRedis(conf)
~~~

这几段代码直接使用了包中的方法



#### 1.ReadConfig方法

~~~go
func ReadConfig(path string) *Config {
	v := viper.New()
	v.SetConfigFile(path)
	v.AutomaticEnv()                                   // 允许使用环境变量
	v.SetEnvKeyReplacer(strings.NewReplacer(".", "_")) // SERVER_APPMODE => SERVER.APPMODE

	if err := v.ReadInConfig(); err != nil {
		panic("配置文件读取失败: " + err.Error())
	}

	if err := v.Unmarshal(&Conf); err != nil {
		panic("配置文件反序列化失败: " + err.Error())
	}

	log.Println("配置文件内容加载成功: ", path)
	return Conf
}
~~~

**函数返回Conf 而Conf又是Config结构体的一个实例化 所以返回的就是Config结构的字段**

所以 **conf := g.ReadConfig(*configPath)**这个函数返回值就是 



#### 2.InitLogger方法

~~~go
// 根据配置文件初始化 slog 日志
func InitLogger(conf *g.Config) *slog.Logger {
	var level slog.Level
	switch conf.Log.Level {
	case "debug":
		level = slog.LevelDebug
	case "info":
		level = slog.LevelInfo
	case "warn":
		level = slog.LevelWarn
	case "error":
		level = slog.LevelError
	default:
		level = slog.LevelInfo
	}

	option := &slog.HandlerOptions{
		AddSource: false, // TODO: 外层
		Level:     level,
		ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
			if a.Key == slog.TimeKey {
				if t, ok := a.Value.Any().(time.Time); ok {
					a.Value = slog.StringValue(t.Format(time.DateTime))
				}
			}
			return a
		},
	}

	var handler slog.Handler
	switch conf.Log.Format {
	case "json":
		handler = slog.NewJSONHandler(os.Stdout, option)
	case "text":
		fallthrough
	default:
		handler = slog.NewTextHandler(os.Stdout, option)
	}

	logger := slog.New(handler)
	slog.SetDefault(logger)
	return logger
}

~~~

#### 3. InitDatabase方法

~~~go

// 根据配置文件初始化数据库
func InitDatabase(conf *g.Config) *gorm.DB {
	dbtype := conf.DbType()
	dsn := conf.DbDSN()

	var db *gorm.DB
	var err error

	var level logger.LogLevel
	switch conf.Server.DbLogMode {
	case "silent":
		level = logger.Silent
	case "info":
		level = logger.Info
	case "warn":
		level = logger.Warn
	case "error":
		fallthrough
	default:
		level = logger.Error
	}

	config := &gorm.Config{
		Logger:                                   logger.Default.LogMode(level),
		DisableForeignKeyConstraintWhenMigrating: true, // 禁用外键约束
		SkipDefaultTransaction:                   true, // 禁用默认事务（提高运行速度）
		NamingStrategy: schema.NamingStrategy{
			SingularTable: true, // 单数表名
		},
	}

	switch dbtype {
	case "mysql":
		db, err = gorm.Open(mysql.Open(dsn), config)
	case "sqlite":
		db, err = gorm.Open(sqlite.Open(dsn), config)
	default:
		log.Fatal("不支持的数据库类型: ", dbtype)
	}

	if err != nil {
		log.Fatal("数据库连接失败", err)
	}
	log.Println("数据库连接成功", dbtype, dsn)

	if conf.Server.DbAutoMigrate {
		if err := model.MakeMigrate(db); err != nil {
			log.Fatal("数据库迁移失败", err)
		}
		log.Println("数据库自动迁移成功")
	}

	return db
}
~~~

#### 4.InitRedis方法

~~~go

// 根据配置文件初始化 Redis
func InitRedis(conf *g.Config) *redis.Client {
	rdb := redis.NewClient(&redis.Options{
		Addr:     conf.Redis.Addr,
		Password: conf.Redis.Password,
		DB:       conf.Redis.DB,
	})

	_, err := rdb.Ping(context.Background()).Result()
	if err != nil {
		log.Fatal("Redis 连接失败: ", err)
	}

	log.Println("Redis 连接成功", conf.Redis.Addr, conf.Redis.DB, conf.Redis.Password)
	return rdb
}

~~~

#### 5.参数 conf  *g.Config

在globle文件下的config.go中定义的结构体Config 并且嵌套一些小部分结构体

~~~go
type Config struct {
	Server struct {
		Mode          string // debug | release
		Port          string
		DbType        string // mysql | sqlite
		DbAutoMigrate bool   // 是否自动迁移数据库表结构
		DbLogMode     string // silent | error | warn | info
	}
	Log struct {
		Level     string // debug | info | warn | error
		Prefix    string
		Format    string // text | json
		Directory string
	}
	JWT struct {
		Secret string
		Expire int64 // hour
		Issuer string
	}
	Mysql struct {
		Host     string // 服务器地址
		Port     string // 端口
		Config   string // 高级配置
		Dbname   string // 数据库名
		Username string // 数据库用户名
		Password string // 数据库密码
	}
	SQLite struct {
		Dsn string // Data Source Name
	}
	Redis struct {
		DB       int    // 指定 Redis 数据库
		Addr     string // 服务器地址:端口
		Password string // 密码
	}
	Session struct {
		Name   string
		Salt   string
		MaxAge int
	}
	Email struct {
		To       string // 收件人 多个以英文逗号分隔 例：a@qq.com,b@qq.com
		From     string // 发件人 要发邮件的邮箱
		Host     string // 服务器地址, 例如 smtp.qq.com 前往要发邮件的邮箱查看其 smtp 协议
		Secret   string // 密钥, 不是邮箱登录密码, 是开启 smtp 服务后获取的一串验证码
		Nickname string // 发件人昵称, 通常为自己的邮箱名
		Port     int    // 前往要发邮件的邮箱查看其 smtp 协议端口, 大多为 465
		IsSSL    bool   // 是否开启 SSL
	}
	Captcha struct {
		SendEmail  bool // 是否通过邮箱发送验证码
		ExpireTime int  // 过期时间
	}
	Upload struct {
		// Size      int    // 文件上传的最大值
		OssType   string // local | qiniu
		Path      string // 本地文件访问路径
		StorePath string // 本地文件存储路径
	}
	Qiniu struct {
		ImgPath       string // 外链链接
		Zone          string // 存储区域
		Bucket        string // 空间名称
		AccessKey     string // 秘钥AK
		SecretKey     string // 秘钥SK
		UseHTTPS      bool   // 是否使用https
		UseCdnDomains bool   // 上传是否使用 CDN 上传加速
	}
}
~~~

初始化结构体

**var Conf  *Config**

使Conf这个对象可以调用调用 一些嵌套结构体中的字段



示例

~~~go
conf.Log.Level
~~~

#### 6.reciver内置函数

`recover` 是一个内建的函数，用于重新获得对 panic 引起的控制权，阻止 panic 继续向上传递。`recover` 可以恢复 panic 发生时的程序执行，通常与 `defer` 语句一起使用。

1. **捕获 panic**：`recover` 可以捕获当前 goroutine 中发生的 panic，并恢复程序的执行。

2. **防止程序崩溃**：通过捕获 panic，可以防止程序因为 panic 而完全崩溃。

   ​                     **注意事项**

3. **只能在 `defer` 中使用**：`recover` 只能在 `defer` 函数中调用，否则它不会捕获到 panic。

4. **不保证总是能捕获**：如果在 `defer` 之前程序已经崩溃，`recover` 可能无法捕获到 panic。

###  八.中间件学习

~~~go
// WithRedisDB 将 redis.Client 注入到 gin.Context
// handler 中通过 c.MustGet(g.CTX_RDB).(*redis.Client) 来使用
func WithRedisDB(rdb *redis.Client) gin.HandlerFunc {
	return func(ctx *gin.Context) {
		ctx.Set(g.CTX_RDB, rdb)
		ctx.Next()
	}
}
~~~

这段代码是一个 **Gin 中间件**，用于将 **Redis 客户端注入到 Gin 的上下文**中。**`WithRedisDB` 函数接受一个 `redis.Client`** 实例，并返回一个 Gin 处理函数。这个处理函数在请求处理过程中将 **Redis 客户端存储在上下文中**，允许后续的处理程序通过 `c.MustGet(g.CTX_RDB).(*redis.Client)` 来访问它。这样可以方便地在不同的处理程序**中共享 Redis 客户端**。



~~~go
  r.Use(middleware.CORS())

  r.Use(middleware.WithGormDB(db))

  r.Use(middleware.WithRedisDB(rdb))

  r.Use(middleware.WithCookieStore(conf.Session.Name, conf.Session.Salt))

  ginblog.RegisterHandlers(r)
~~~



1. `middleware.CORS()`：处理跨源资源共享，允许不同源的请求访问你的 API。
2. `middleware.WithGormDB(db)`：注入 GORM 数据库连接，方便后续处理程序访问数据库。
3. `middleware.WithRedisDB(rdb)`：注入 Redis 客户端，允许处理程序访问 Redis 数据。
4. `middleware.WithCookieStore(...)`：设置会话管理，使用 Cookie 存储用户会话信息。

这些中间件的组合使得你的应用能够处理 HTTP 请求时具备更强的功能和灵活性。



~~~go
	docs.SwaggerInfo.BasePath = "/api"
~~~

- `docs.SwaggerInfo` 是一个 Swagger 文档的结构体，包含了文档的各种配置信息。
- `BasePath` 属性设置了 API 的基础路径，在这个例子中是 `"/api"`。

这意味着所有 API 的路径都会以 `/api` 开头。例如，如果你有一个 GET 请求的路径是 `/users`，最终的完整路径就是 `/api/users`。这样做可以帮助组织 API 路由并保持一致性。



###  九.manage.go文件

~~~go

// 通用接口: 全部不需要 登录 + 鉴权
func registerBaseHandler(r *gin.Engine) {
	base := r.Group("/api")

	// TODO: 登录, 注册 记录日志
	base.POST("/login", userAuthAPI.Login)          // 登录
	base.POST("/register", userAuthAPI.Register)    // 注册
	base.GET("/logout", userAuthAPI.Logout)         // 退出登录
	base.POST("/report", blogInfoAPI.Report)        // 上报信息
	base.GET("/config", blogInfoAPI.GetConfigMap)   // 获取配置
	base.PATCH("/config", blogInfoAPI.UpdateConfig) // 更新配置
	base.GET("/code", userAuthAPI.SendCode)         // 验证码
}
~~~



**swagger ui**



~~~go
"golang.org/x/net/webdav"
WebDAV 服务器或相关的文件操作功能
~~~

### 十.cmd中 create-superadmin文件夹

**运行程序时传入参数**

**观察.sh文件 在其中描述了 指令的使用例如**

~~~sh
cd generate-data

# all | config | auth | page | resource
# all 生成所有信息
# config 生成配置信息
# auth 生成默认角色 admin, guest, 以及对应的默认用户 admin, guest
# page 生成默认页面信息
# resource 生成默认资源信息
go run main.go -t "all"
~~~

~~~sh
cd create-superadmin

# 创建一个用户名 superadmin, 密码 superadmin 的超级管理员
# 超级管理员可以访问所有页面, 所有资源
go run main.go -username superadmin -password superadmin
~~~











~~~shell
go run main.go --username=zhaozhonghe --password="zzh123456"
~~~

~~~shell
024/09/26 16:42:02 配置文件内容加载成功:  ../../config.yml
2024/09/26 16:42:02 数据库连接成功 mysql root:123456@tcp(127.0.0.1:3306)/gvb?charset=utf8mb4&parseTime=True&loc=Local
2024/09/26 16:42:02 数据库自动迁移成功
2024/09/26 16:42:02 INFO 开始创建超级管理员
2024/09/26 16:42:02 INFO 创建超级管理员成功: zhaozhonghe, 密码: zzh123456
~~~

这段代码意思是：

~~~go
dsn := conf.DbDSN()
~~~

输出

~~~shell
username:password@tcp(host:port)/dbname
~~~



### 十一. 路由 /api/config



~~~go

func GetConfigMap(db *gorm.DB) (map[string]string, error) {
	var configs []Config
	result := db.Find(&configs)
	if result.Error != nil {
		return nil, result.Error
	}

	m := make(map[string]string)
	for _, config := range configs {
		m[config.Key] = config.Value
	}

	return m, nil
}
~~~







~~~go

func (*BlogInfo) GetConfigMap(c *gin.Context) {
	db := GetDB(c)
	rdb := GetRDB(c)

	// get from redis cache
	cache, err := getConfigCache(rdb)
	if err != nil {
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	if len(cache) > 0 {
		slog.Debug("get config from redis cache")
		ReturnSuccess(c, cache)
		return
	}

	// get from db
	data, err := model.GetConfigMap(db)
	if err != nil {
		ReturnError(c, g.ErrDbOp, err)
		return
	}

	// add to redis cache
	if err := addConfigCache(rdb, data); err != nil {
		ReturnError(c, g.ErrRedisOp, err)
		return
	}

	ReturnSuccess(c, data)
}
~~~



对应返回数据

~~~json
{
    "code": 0,
    "message": "OK",
    "data": {
        "article_cover": "https://cdn.hahacode.cn/config/default_article_cover.png",
        "gitee": "https://gitee.com/szluyu99",
        "github": "https://github.com/szluyu99",
        "is_comment_review": "true",
        "is_message_review": "true",
        "qq": "123456789",
        "tourist_avatar": "https://cdn.hahacode.cn/config/tourist_avatar.png",
        "user_avatar": "https://cdn.hahacode.cn/config/user_avatar.png",
        "website_author": "阵雨",
        "website_avatar": "https://foruda.gitee.com/avatar/1677041571085433939/5221991_szluyu99_1614389421.png",
        "website_createtime": "2024-09-21 19:00:39",
        "website_intro": "往事随风而去",
        "website_name": "阵雨的个人博客",
        "website_notice": "欢迎来到阵雨的个人博客，项目还在开发中...",
        "website_record": "粤ICP备2021032312号"
    }
}
~~~

