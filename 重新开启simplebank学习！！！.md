# 重新开启simplebank学习！！！



### 一.创建数据库表

https://dbdiagram.io 可视化数据库工具

#### 1.创建账户表

~~~go
Table accounts as A { //A作为account的别名
  id bigserisal [pk]  //pk作为主键 自增的id列
  owner varchar
  balance bigint
  currency varchar 
  created_at timestamp [default: `now()`] //自动获取时间
}
~~~

#### 2.创建条目表 

//记录账户余额的变化

~~~go
Table entries {
  id bigint [pk] //
  account_id bigint [ref : > A.id] //外键 账户和条目之间是1对多关系
    amount bigint [not null note:`可以是负或者正`] //正负取决于取出还是存入 note是添加注释
  created_at timestamp [default: `now()`] //记录条目的创建时间
}
~~~

#### 3.创建 转账表

~~~go
Table transfers {
  id bigint [pk]
  from_account_id bigint [ref : > A.id]
  to_account_id bigint [ref : > A.id]
    amount  bigint [not null note: `一定不能为空`]//note为注释
  created_at timestamp [default: `now()`]
}
~~~

在此之后向列中添加非空约束 例如 ：

 balance bigint [not null] //  **非空约束是一种用于限制数据库表中某列不能为空的约束**



枚举

~~~
Enum Currency{
USD
EUR
}
~~~



向表中添加索引

~~~go
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Table accounts as A {
  id bigserisal [pk]
  owner varchar [not null]
  balance bigint [not null]
  currency varchar  [not null]
  created_at timestamp [default: `now()`] 

  Indexes {
    owner
  }
}

Table entries {
  id bigint [pk]
  account_id bigint [ref : > A.id] 
  
  amount bigint [not null]
  created_at timestamp [default: `now()`]
  
 //列出特定账户的所有条目
Indexes {
  account_id
}
                        
}

Table transfers {
  id bigint [pk]
  from_account_id bigint [ref : > A.id]
  to_account_id bigint [ref : > A.id]
  amount  bigint [not null]
  created_at timestamp [default: `now()`]

  
Indexes {
  from_account_id
  to_account_id
  (from_account_id,to_account_id)
}
}

~~~

这些做好之后使用导出功能 生成代码

~~~go
CREATE TABLE "accounts" (
  "id" bigserisal PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigint PRIMARY KEY,
  "account_id" bigint,
  "amount" bigint NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigint PRIMARY KEY,
  "from_account_id" bigint,
  "to_account_id" bigint,
  "amount" bigint NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

//将外键添加到表中

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");


~~~



### 二.Docker

**用指令创建容器的时候 一定要注意-p参数 将容器的端口映射到主机上 一定要保证端口不要被占用 否则将会产生问题**



拉取镜像语法

~~~shell
docker pull <image>:<tag>
~~~

开始一个容器指令

~~~shell
1.docker run --name <container_name> -e <environment_variable> -d <image>:tag
:
2.docker run --name some-postgres -e POSTGRES_PASSWORD=mysecret -d postgres 
#！！！端口映射 -p 5432:5432 //注意防止端口冲突自行更改
示例：
docker run --name postgres12 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine 

3. docker exec -it <contain_name_or_id> <comman> [args]
示例：#进入psql控制台
docker exec -it postgres12 psql -U root

4.显示容器日志
docker logs <container_name_or_id>
示例:
docker logs postgres12

5.连接shell
指令：
docker exec -it postgres12 /bin/sh
#创建新的数据库
createdb --username=root --owner=root simple_bank
#使用psql连接
 psql simple_bank
 #删除数据库
 dropdb [名称]
 
 exit退出shell
 
 #指令结合
 docker exec -it postgres12 createdb --username=root --owner=root simple_bank
 docker exec -it postgres12 psql -U root simple_bank
 
 #查找指令
 history | grep "docker run" //linux
 history | Select-String "docker run"//windows
 
~~~

区分 docker中的 镜像和容器

docker image中包含多个运行 容器的应用实例 类似结构：

\- docker image

- ├── container1   

* ├──container2

* ├──container3

### 三.Tableplus

将sql文件导入到tableplus中

在tableplus中删除表 使用sql指令

~~~sql
DROP TABLE accounts CASCADE; //注意替换表名称
~~~





### 四.DB migration

迁移指令：

~~~
 migrate create -ext sql -dir db/migration -seq init_schema
~~~



up/down migration：理解迁移 类比栈结构 向上新数据表 向下 旧数据表

**使用migrate up指令时         Old DB 在文件中 一次按照 1.up.sql 2.up.sql 3.up.sql 依次运行到New DB**

**使用migrate down指令时  New DB 在文件中依次按照 3.up.sql 2.up.sql  1.up.sql 依次运行到Old DB**



**old DB schema** —–> migrate up —––> x.up.sql —–>**New DB schema**

​      <—————————-  x.down.sql <———migrate down<————-		



将最开始的.sql文件放入 .up.sql中

### 五.Makefile文件

**创建规则后使用  make指令 快速创建**

如果你是萌新开始给到你一个项目 你可以通过makefile文件快速构建

~~~shell
migrate -help
#通过看日志 知道使用什么指令来工作
#迁移指令
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank" -verbose up
#出现ssl错误
添加sslmode=disabled

#出现了一系列的迁移错误   解决方案
强制更改版本
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose force 1
~~~



### 六.数据库的CRUD

DATAVASE/SQL库 

GORM

sqlx（兼容多）

sqlc（最好的 融合了以上两者的优点）



### 七.使用sqlc

~~~
sqlc init
~~~

介绍:

sqlc 从 SQL 生成**类型安全的代码**。以下是它的工作原理：

1. 您使用 SQL 编写查询。
2. 运行 sqlc 来生成具有这些查询的类型安全接口的代码。
3. 编写调用生成的代码的应用程序代码。

查看[一个交互式示例](https://play.sqlc.dev/)来了解它的实际应用，以及 sqlc 背后的动机的[介绍性博客文章](https://conroy.org/introducing-sqlc)。



### 八.sqlc.yaml

~~~go
version: "2"
sql:
- schema: "simplebank/db/migration" //数据库表
  queries: "db/query" //数据库查询 首先要编写数据库查询
  engine: "postgresql" //使用的数据库
  gen:
    go: 
      package: "db"
      out: "simplebank/db/sqlc"
      sql_package: "pgx/v5"
      emit_json_tags: true
      emit_interface: false
      emit_empty_slices: true
      overrides:
        - db_type: "timestamptz"
          go_type: "time.Time"
        - db_type: "uuid"
          go_type: "github.com/google/uuid.UUID"
~~~

account.sql

~~~sql
-- name: CreateAccount :one
INSERT INTO accounts (
  owner,
  balance,
  currency
) VALUES (
  $1, $2, $3
) RETURNING *;
~~~



#### make sqlc 生成代码

##### 1.account.sql.go

##### 2.db.go

##### 3.models.go

**在生成之后由于没有 初始化项目 使得项目报红**

~~~shell
go mod init project/simplebank
go mod tidy
~~~





### 九.编写单元测试用例

1.导入未使用的包在前面添加_可以防止系统自动将它删除

例如：

_ "github.com/lib/pq "

错误

~~~
cannot use conn (variable of type *sql.DB) as DBTX value in argument to New: *sql.DB does not implement DBTX (wrong type for method Exec)
		have Exec(string, ...any) (sql.Result, error)
		want Exec(context.Context, string, ...interface{}) (pgconn.CommandTag,
		
        你遇到的错误信息表明，你尝试将 *sql.DB 类型的 conn 变量用作 DBTX 类型的参数，但 *sql.DB 并不满足 DBTX 接口的要求。具体来说，DBTX 接口期望的 Exec 方法签名是 (context.Context, string, ...interface{}) (pgconn.CommandTag, error)，而 *sql.DB 提供的 Exec 方法签名是 (string, ...any) (sql.Result, error)。
~~~



**目前不理解  方法和接口 开始学习**

~~~go
//这是接口
type DBTX interface {
	Exec(context.Context, string, ...interface{}) (pgconn.CommandTag, error)
	Query(context.Context, string, ...interface{}) (pgx.Rows, error)
	QueryRow(context.Context, string, ...interface{}) pgx.Row
}

string, ...interface{}//的含义这是可变参数的声明方式。它表示可以接收零个或多个任意类型的参数。在函数内部，这些参数被视为一个 []interface{}（一个接口类型的切片）。
例如，如果有一个函数接受一个字符串和任意数量的其他参数：

func myFunction(s string, args...interface{}) {
    // 函数体
}

可以这样调用这个函数：
myFunction("hello", 1, "world", true)




/*
 在这段代码中有一个这样的错误我不知道怎么修改cannot use conn (variable of type *sql.DB) as DBTX value in argument to New: *sql.DB does not implement DBTX (wrong type for method Exec) have Exec(string, ...any) (sql.Result, error) want Exec(context.Context, string, ...interface{}) (pgconn.CommandTag,



要解决这个问题，你需要确保你的 DBTX 接口的方法签名与 *sql.DB 的实现匹配。具体来说，你需要修改 DBTX 接口的 Exec 方法，使其接受 context.Context 参数。以下是一个可能的解决方案：

修改 DBTX 接口，使其 Exec 方法的签名为：

type DBTX interface {
    Exec(ctx context.Context, query string, args ...interface{}) (pgconn.CommandTag, error)
    // 其他方法...
}
在你的 New 函数中，确保它接受符合 DBTX 接口的类型。例如，使用 sql.DB 时，你可以创建一个适配器来实现 DBTX 接口。
这里是一个适配器示例：


type DBAdapter struct {
    *sql.DB
}

func (db *DBAdapter) Exec(ctx context.Context, query string, args ...interface{}) (pgconn.CommandTag, error) {
    result, err := db.DB.ExecContext(ctx, query, args...)
    return result, err
}
然后在 TestMain 中使用这个适配器：

testQueries = New(&DBAdapter{conn})
这样就能确保 *sql.DB 能正确实现 DBTX 接口，从而解决你遇到的错误。
*/

!!!!!!!!!!!!!!!!!!!!!!gpt给出解决问题的方法在我不知道用pgx的情况下 gpt给出用pgx连接
给gpt看完接口定义之后
由于你的 DBTX 接口使用了 pgx 库，你可以使用 pgx 的 Conn 类型而不是 *sql.DB。这里是一个简单的解决方案：
使用 pgx 连接：确保在代码中使用 pgx 的连接，而不是 *sql.DB。修改你的数据库连接部分：
go
复制代码
import (
	"context"
	"log"
	"os"
	"testing"

	"github.com/jackc/pgx/v4"
)

const (
	dbDriver = "pgx"
	dbSource = "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable"
)

var testQueries *Queries

func TestMain(m *testing.M) {
	conn, err := pgx.Connect(context.Background(), dbSource)
	if err != nil {
		log.Fatal("cannot connect to db:", err)
	}
	testQueries = New(conn) // 确保 New 函数接收 pgx.Conn

	os.Exit(m.Run())
}
确保 New 函数接受 pgx.Conn：在你的 New 函数中，确保它接受 pgx.Conn 而不是 *sql.DB。
这样，你的 DBTX 接口与 pgx 的连接就能正常工作，避免了类型不匹配的问题。这样也简化了代码，直接使用了 pgx 连接而不需要适配器。



~~~

---



#### 10.8日 解决目前为止遇见的一个问题





*遇见的第一个大问题 在account_test.go中和main_test.go中出现的一些问题*

*和姐聊了一下 解决了 **testQueries = New(conn)**中的问题*  

*main_test.go使用 pgx来连接数据库 而不是视频中讲解的sql.open 他们的返回值类型不同造成了错误*

测试函数的拼写错误 可能也影响了一大部分  



还有最后一处问题

1. `require.NotEmpty(t, err)`这一行存在问题。这里应该是检查`account`是否不为空，而不是检查错误`err`是否不为空。正确的应该是`require.NotEmpty(t, account)`。



不熟悉的地方go语言的包 接口 方法 

----



#### 1.最终的account_test.go代码

~~~go
package db

import (
	"context"
	"testing"

	"github.com/stretchr/testify/require"
)

func TestCreateAccount(t *testing.T) {
	arg := CreateAccountParams{
		Owner:    "xiaozhao",
		Balance:  100,
		Currency: "USD",
	}

	account, err := testQueries.CreateAccount(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, account)

	require.Equal(t, arg.Owner, account.Owner)
	require.Equal(t, arg.Balance, account.Balance)
	require.Equal(t, arg.Currency, account.Currency)

	require.NotZero(t, account.ID)
	require.NotZero(t, account.CreatedAt)
}

~~~



#### 2.最终的main_test.go代码

~~~go
package db

import (
	"context"
	"fmt"
	"os"
	"testing"

	"github.com/jackc/pgx/v5"
)

var testQueries *Queries

const (
	DATABASE_URL = "postgres://root:secret@localhost:5432/simple_bank?sslmode=disable"
)

func TestMain(m *testing.M) {

	conn, err := pgx.Connect(context.Background(), DATABASE_URL)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer conn.Close(context.Background())

	testQueries = New(conn)
	os.Exit(m.Run())
}

//测试通过！！！！！！！
~~~



**上面是指定了一个一个账户 我们想让账户的主人 货币 钱是随机的 编写util中的random代码**：

~~~go
package util

import (
    "math/rand"
    "strings"
    "time"
)

const alphabet = "abcdefghijklmnopqrstuvwxyz"

var rng *rand.Rand

func init() {
    source := rand.NewSource(time.Now().UnixNano())
    rng = rand.New(source)
}

// 返回一个介于 min max 之间的随机的 int64 数字
func RandomInt(min, max int64) int64 {
    return min + rng.Int63n(max-min+1)
}

// 生成 n 个字符的随机字符串
func RandomString(n int) string {
    var sb strings.Builder
    k := len(alphabet)

    for i := 0; i < n; i++ {
        c := alphabet[rng.Intn(k)]
        sb.WriteByte(c)
    }

    return sb.String()
}


//随机生成owner
func RandomOwner() string{
	return RandomString(6)
}

//随机生成钱的数量
func RandomMoney() int64{
	return RandomInt(0,1000 )
}

//随机产生一种货币
func RandowCurrency() string{
	currencies := []string{"RMB","USD","CAD"}
	n := len(currencies)
	return currencies[rand.Intn(n)]
}
~~~

**学会如何把自己写的包导入到别的文件夹下  这个需要看go mod下的 module project/simplebank  **

**把moudle中的包作为起始路径 导入到别的文件夹下  就是："project/simplebank/util"**



ok 截止到 10.8日随机生成的数据生成功

---



**问题2**

makefile文件中的下面这个指令

**test:**

  **go test -v -cover ./…     这个指令必须在当前目录下找到go的测试文件**

**就是go.mod文件应该和makefile保持在一起**    解决方法在本地的go.mod文件夹下又创建了一个makefile 用来测试 make test



类型断言 interface代表为止类型 使用前需要 转换为具体类型   (从未知类型转为已知类型)

如：

~~~
var i interface{} = 2
num1， ok ：= i.(int)//断言
~~~

---



#### 3.account_test.go代码：

全部测试通过！

~~~go
package db

import (
	"context"

	"project/simplebank/util"
	"testing"
	"time"

	"github.com/stretchr/testify/require"
)

func createRandomAccount(t *testing.T) Account {
	arg := CreateAccountParams{
		Owner:    util.RandomOwner(),
		Balance:  util.RandomMoney(),
		Currency: util.RandomCurrency(),
	}

	account, err := testQueries.CreateAccount(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, account)

	require.Equal(t, arg.Owner, account.Owner)
	require.Equal(t, arg.Balance, account.Balance)
	require.Equal(t, arg.Currency, account.Currency)

	require.NotZero(t, account.ID)
	require.NotZero(t, account.CreatedAt)

	return account
}

func TestCreateAccount(t *testing.T) {

	createRandomAccount(t)
}

func TestGetAccount(t *testing.T) {
	account1 := createRandomAccount(t)
	account2, err := testQueries.GetAccount(context.Background(), account1.ID)

	require.NoError(t, err)
	require.NotEmpty(t, account2)

	require.Equal(t, account1.ID, account2.ID)
	require.Equal(t, account1.Owner, account2.Owner)
	require.Equal(t, account1.Balance, account2.Balance)
	require.Equal(t, account1.Currency, account2.Currency)

	require.WithinDuration(t, account1.CreatedAt.Time, account2.CreatedAt.Time, time.Second)
}

func TestUpdateAccount(t *testing.T) {
	account1 := createRandomAccount(t)

	arg := UpdateAccountParams{
		ID:      account1.ID,
		Balance: util.RandomMoney(),
	}
	account2, err := testQueries.UpdateAccount(context.Background(), arg)

	require.NoError(t, err)
	require.NotEmpty(t, account2)

	require.Equal(t, account1.ID, account2.ID)
	require.Equal(t, account1.Owner, account2.Owner)
	require.Equal(t, arg.Balance, account2.Balance)
	require.Equal(t, account1.Currency, account2.Currency)

	require.WithinDuration(t, account1.CreatedAt.Time, account2.CreatedAt.Time, time.Second)
}

func TestDeleteAccount(t *testing.T) {
	account1 := createRandomAccount(t)
	err := testQueries.DeleteAccount(context.Background(), account1.ID)
	require.NoError(t, err)

	account2, err := testQueries.GetAccount(context.Background(), account1.ID)
	require.Error(t, err)
	//	require.EqualError(t, err, sql.ErrNoRows.Error())
	require.Empty(t, account2)
}

func TestListAccount(t *testing.T) {
	var lastAccount Account
	for i := 0; i < 10; i++ {
		lastAccount = createRandomAccount(t)
	}
	arg := ListAccountsParams{
		Owner:  lastAccount.Owner,
		Limit:  5, //返回五条记录
		Offset: 0, //设置偏移量 返回后五条记录  这里出现了问题！！！！
	}
	accounts, err := testQueries.ListAccounts(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, accounts)

	for _, account := range accounts {
		require.NotEmpty(t, account)
		require.Equal(t, lastAccount.Owner, account.Owner)
	}
}

~~~

---



#### 4.entry_test.go

条目上的account.id要和account表单上的di相对应



**问题**



**id为null**

***在 PostgreSQL 中，如果一个表的 `id` 字段没有设置为自增序列（如 `bigserial`），并且你在插入数据时没有显式地为 `id` 字段指定值，那么 `id` 字段的值将会是 `NULL`，除非该字段设置了默认值。***

**解决办法**

~~~sql
创建一个序列：首先，你需要创建一个序列，这个序列将用于生成 id 列的值。
   CREATE SEQUENCE entries_id_seq;
~~~

~~~sql
设置序列的所有权：将序列与 id 列关联起来。
   ALTER SEQUENCE entries_id_seq OWNED BY entries.id;
~~~

~~~sql
设置 id 列的默认值为序列的下一个值：这样，每当你插入新行而没有指定 id 值时，PostgreSQL 会自动使用序列的下一个值。
   ALTER TABLE entries ALTER COLUMN id SET DEFAULT nextval('entries_id_seq');
 
~~~

  **4确保 `id` 列是主键**：从你提供的信息来看，`id` 列已经是主键。确保这一点很重要，因为主键约束可以保证 `id` 列的值是唯一的。

~~~sql
测试：插入一条新记录，不指定 id 值，检查是否自动生成了 id。
   INSERT INTO entries (account_id, amount, created_at) VALUES (1, 100, now());
~~~

~~~sql
DELETE FROM entries WHERE id=4; 删除特定行的指令
~~~



创建账单成功！



但是只能生成一个数据？？？

我发现了输出的区别 Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestCreateEntry$ project/simplebank/db/sqlc

ok  	project/simplebank/db/sqlc	(cached) 这是第二次输出        第一次输出没有cached字样 数据正确的加载到了数据库 但是这个带有cached的数据没有加载到数据库



**因为 cached 是因为两次的数据相同 所以才没有被加载到数据库 这个可能是随机数代码的问题**

---

#### 5.transfer_test.go

~~~go
package db

import (
	"context"
	"time"

	"project/simplebank/util"
	"testing"

	"github.com/stretchr/testify/require"
)

func createRandomTransfer(t *testing.T, account1, account2 Account) Transfer {
	arg := createTransferParams{
		FromAccountID: account1.ID,
		ToAccountID:   account2.ID,
		Amount:        util.RandomMoney(),
	}
	transfer, err := testQueries.createTransfer(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, transfer)

	require.Equal(t, arg.FromAccountID, transfer.FromAccountID)
	require.Equal(t, arg.ToAccountID, transfer.ToAccountID)
	require.Equal(t, arg.Amount, transfer.Amount)

	require.NotZero(t, transfer.ID)
	require.NotZero(t, transfer.CreatedAt)

	return transfer
}

func TestCreateTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)
	createRandomTransfer(t, account1, account2)
}

func TestGetTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)
	transfer1 := createRandomTransfer(t, account1, account2)

	transfer2, err := testQueries.GetTransfer(context.Background(), transfer1.ID)
	require.NoError(t, err)
	require.NotEmpty(t, transfer2)

	require.Equal(t, transfer1.ID, transfer2.ID)
	require.Equal(t, transfer1.FromAccountID, transfer2.FromAccountID)
	require.Equal(t, transfer1.ToAccountID, transfer2.ToAccountID)
	require.Equal(t, transfer1.Amount, transfer2.Amount)
	require.WithinDuration(t, transfer1.CreatedAt.Time, transfer2.CreatedAt.Time, time.Second)
}

func TestListTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)

	for i := 0; i < 5; i++ {
		createRandomTransfer(t, account1, account2)
		createRandomTransfer(t, account2, account1)
	}

	arg := ListTransfersParams{
		FromAccountID: account1.ID,
		ToAccountID:   account1.ID,
		Limit:         5,
		Offset:        5,
	}

	transfers, err := testQueries.ListTransfers(context.Background(), arg)
	require.NoError(t, err)
	require.Len(t, transfers, 5)

	for _, transfer := range transfers {
		require.NotEmpty(t, transfer)
		require.True(t, transfer.FromAccountID == account1.ID || transfer.ToAccountID == account1.ID)
	}
}

~~~



### 十.db transaction

  

BEGIN语句启动事务

成功 则更新数据库

失败 则回滚事务（保持原来的状态）

​                                                                                                    

代码对不上了 决定先复制粘贴 学习数据库中的知识点





**先从config.go开始**

****

### 十一.config.go

**使用viper**

创建app.env文件存储配置信息

config.go

~~~go
package util

import (
	"github.com/spf13/viper"
)

// Config stores all configuration of the application.
// The values are read by viper from a config file or environment variable.
type Config struct {
	DATABASE_URL string `mapstructure:"DATABASE_URL"`
}

// LoadConfig reads configuration from file or environment variables.
func LoadConfig(path string) (config Config, err error) {
	viper.AddConfigPath(path)
	viper.SetConfigName("app")
	viper.SetConfigType("env")

	viper.AutomaticEnv()

	err = viper.ReadInConfig()
	if err != nil {
		return
	}

	err = viper.Unmarshal(&config)
	return
}
~~~

---



使用接口来简化一些操作好好学接口

目前为止更正了大部分问题接着往下学。。。。

**store.test.go出现了大问题**

**报错：**

**Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc >> before: 1984 3906 --- FAIL: TestTransferTx (0.03s)    e:\projects\simplebank\db\sqlc\store_test.go:83:         	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:83        	Error:      	Should NOT be empty, but was {0  0  {0001-01-01 00:00:00 +0000 UTC finite false}}        	Test:       	TestTransferTx FAIL FAIL	project/simplebank/db/sqlc	****0.571s**



因为还没编写代码。。。。。草草草草操操操操哦哦操操操这视频叫我看的

---

### 十二.需要仔细处理并发 交易 以避免死锁

数据库事务  

~~~shell
Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc

>> before: 6892 6969
>> tx: 6882 6979
>> tx: 6882 6989
--- FAIL: TestTransferTx (0.04s)
    e:\projects\simplebank\db\sqlc\store_test.go:102: 
        	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:102
        	Error:      	Not equal: 
        	            	expected: 10
        	            	actual  : 20
        	Test:       	TestTransferTx
FAIL
FAIL	project/simplebank/db/sqlc	0.602s
FAIL

~~~

这个问题出在

account.sql.go他无法阻止一些东西

~~~
-- name: GetAccount :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1;
~~~

**在两个终端中并行运行两个事务来观察这个问题**

*BEGIN；开始事务*

*ROLLBACK；回滚事务*

：第一个终端

~~~~shell
simple_bank=# BEGIN;
BEGIN
simple_bank=# BEGIN;
WARNING:  there is already a transaction in progress
BEGIN
simple_bank=# ROLLBACK;
ROLLBACK
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     100 | USD      | 2024-10-08 09:03:03.272176
(1 row)
~~~~

第二个终端

~~~shell

simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;

这里会被阻止 并且必须等待第一个事务提交或回滚
~~~



纠正方法1： 在sql中添加 ： 重新用make sqlc生成

~~~sql
-- name: GetAccountForUpdate :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1
FOR UPDATE;
~~~

但是接下来出现了死锁错误：

添加日志寻找错误

~~~
Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc

>> before: 1826 5993
tx 5 Create transfer
tx 5 Create entry 1
tx 5 Create entry 2
tx 5 get account 1
tx 2 Create transfer
tx 5 update account 1
tx 5 get account 2
tx 4 Create transfer
tx 5 update account 2
tx 3 Create transfer
tx 2 Create entry 1
tx 4 Create entry 1
tx 3 Create entry 1
tx 1 Create transfer
tx 2 Create entry 2
tx 4 Create entry 2
tx 3 Create entry 2
tx 2 get account 1
tx 4 get account 1
tx 3 get account 1
>> tx: 1816 6003
tx 1 Create entry 1
tx 1 Create entry 2
tx 1 get account 1
--- FAIL: TestTransferTx (0.95s)
    e:\projects\simplebank\db\sqlc\store_test.go:52: 
        	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:52
        	Error:      	Received unexpected error:
        	            	ERROR: deadlock detected (SQLSTATE 40P01)
        	Test:       	TestTransferTx
FAIL
FAIL	project/simplebank/db/sqlc	1.470s
FAIL

~~~



终端事务出现错误：

~~~sql
INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *;
~~~



1. - 第一个错误 `INSERT INFO transfers` 是语法错误，正确的语法是 `INSERT INTO transfers`。
   - 第二个错误 `INSERT INTO transfers (from_account_id to_account_id amount)` 也存在语法错误，缺少逗号分隔列名。正确的写法是 `INSERT INTO transfers (from_account_id, to_account_id, amount)`。
2. **当前事务已中止**：
   - 由于之前的 SQL 语句（可能是第一条插入语句）出错，事务被标记为 "aborted"。这意味着在该事务中的所有后续 SQL 命令都将失败，直到事务被回滚。

### 解决方法

1. **结束当前事务**：

   - 在 PostgreSQL 中，你可以通过以下命令结束当前事务并回滚更改：

     ~~~sql
     ROLLBACK
     ~~~

#### 终端阻塞 事务状态



~~~
1. 确认当前事务状态
在 PostgreSQL 中，如果一个事务因为某种原因（例如错误或未处理的异常）而中断，那么所有后续的 SQL 语句将会被忽略，直到你执行 ROLLBACK 或 COMMIT。首先，确保没有事务在进行中。

你可以使用以下命令查看当前活动的事务：

SELECT * FROM pg_stat_activity WHERE state = 'active';
~~~

~~~sql
simple_bank=# INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *;
^CCancel request sent
ERROR:  canceling statement due to user request
CONTEXT:  SQL statement "SELECT 1 FROM ONLY "public"."accounts" x WHERE "id" OPERATOR(pg_catalog.=) $1 FOR KEY SHARE OF x"
simple_bank=# SELECT * FROM pg_stat_activity WHERE state = 'active';
 datid |   datname   | pid | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |         state_change
 | wait_event_type |  wait_event   | state  | backend_xid | backend_xmin |                                            query                                            |  backend_type
-------+-------------+-----+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+-----------------+---------------+--------+-------------+--------------+---------------------------------------------------------------------------------------------+----------------
 16385 | simple_bank | 810 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:58:51.115963+00 | 2024-10-10 13:00:09.094842+00 | 2024-10-10 13:00:09.094842+00 | 2024-10-10 13:00:09.094867+00 |                 |               | active |             |          935 | SELECT * FROM pg_stat_activity WHERE state = 'active';                                      | client backend
 16385 | simple_bank | 802 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:58:07.854964+00 | 2024-10-10 12:58:09.850035+00 | 2024-10-10 12:58:09.850035+00 | 2024-10-10 12:58:09.850039+00 | Lock            | tuple         | active |         965 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 782 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:51:58.685461+00 | 2024-10-10 12:57:47.014409+00 | 2024-10-10 12:57:47.014409+00 | 2024-10-10 12:57:47.014413+00 | Lock            | tuple         | active |         963 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 775 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:51:51.971697+00 | 2024-10-10 12:52:16.478898+00 | 2024-10-10 12:57:54.280622+00 | 2024-10-10 12:57:54.280627+00 | Lock            | tuple         | active |         964 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 556 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 11:29:44.776508+00 | 2024-10-10 11:56:11.909182+00 | 2024-10-10 11:56:24.551023+00 | 2024-10-10 11:56:24.551027+00 | Lock            | transactionid | active |             |          935 | SELECT * FROM accounts WHERE id=1 FOR UPDATE;                                               | client backend
 16385 | simple_bank | 638 |       10 | root    | psql          
~~~

---

从你提供的 `pg_stat_activity` 的查询结果来看，有多个活跃的事务正在等待锁。这可能是导致你无法执行 `INSERT` 操作的原因。以下是一些步骤，可以帮助你诊断并解决这个问题：

#### 1. **识别并终止阻塞的事务**

你的查询结果显示了多个 `INSERT` 操作在等待锁。可以尝试强制终止这些阻塞的事务，以释放锁并允许你的操作继续进行。首先，记录下阻塞的 `pid`（进程 ID），然后使用以下命令终止它们：

```
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775);
```

在这个例子中，`802`、`782` 和 `775` 是阻塞的 `pid`。请根据你的实际情况调整这条命令。

#### 2. **确认事务的状态**

如果终止了这些连接，确保你没有其他活跃的事务在运行。你可以再次运行以下命令确认：

```
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

#### 3. **尝试再次执行插入**

在确保没有其他事务阻塞后，尝试重新执行你的插入操作：

```sql
BEGIN;  -- 开始新的事务
INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1, 2, 10) RETURNING *;  -- 执行插入
COMMIT;  -- 提交事务

指令；
SELECT * FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accounts SET balance = 600 WHERE id=1;
```

#### 4. **如果问题依然存在**

如果问题依然存在，建议检查数据库的锁状态：

```
SELECT * FROM pg_locks WHERE granted = false;
```

**又产生错误**

~~~
simple_bank=# SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775,556,638,810);
FATAL:  terminating connection due to administrator command
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
~~~

回答：

看起来你在尝试终止阻塞的连接时，PostgreSQL 服务器意外关闭了连接。这可能是由于以下几个原因：

1. **服务器崩溃**：可能是因为在终止连接时，某个后台进程遇到了问题，导致服务器崩溃。
2. **资源不足**：如果你的 PostgreSQL 服务器资源不足（例如内存或 CPU），在处理请求时可能会出现这种情况。

~~~
由于服务器崩溃，重启 Docker 容器可能会帮助你恢复连接：

docker restart postgres12
在重启后检查连接：

重启后，尝试重新连接到数据库，并检查活动连接：

SELECT * FROM pg_stat_activity;
再次终止阻塞的连接：

如果连接正常，尝试再次运行终止命令：

SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775, 
~~~



---



**git上传一个项目没有共同历史**

~~~shell
检查是否有共同历史
git log --oneline --graph --all
~~~

~~~shell
强制合并冲突
git pull origin main --allow-unrelated-histories
~~~

---



终端1：在没有阻塞的情况下

~~~sql

simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     100 | USD      | 2024-10-08 09:03:03.272176
(1 row)

simple_bank=# UPDATE accounts SET balance = 600 WHERE id=1;
UPDATE 1
simple_bank=# COMMIT;
COMMIT
simple_bank=#

~~~

在终端一提交事务时 终端二会显示出结果

~~~sql
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     600 | USD      | 2024-10-08 09:03:03.272176
(1 row)

simple_bank=#


~~~



sql QUERIER

~~~sql
BEGIN;

INSERT INTO transfers (from_account_id,to_account_id,amount) VALUE (1,2,10) RETURNING *;

INSERT INTO entries (account_id,amount) VALUES (1,-10) RETURNING *;
INSERT INTO entries (account_id,amount) VALUES (2,10) RETURNING *;

SELECT * FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accounts SET balance = 90 WHERE id = 1 RETURNING *;

SELECT * FROM accounts WHERE id =2 FOR UPDATE;
UPDATE accounts SET balance = 110 WHERE id = 2 RETURNING *;

ROLLBACK;
~~~

---



#### postgres lock：帮助查询哪里有锁

The following query may be helpful to see what processes are blocking SQL statements (these only find row-level locks, not object-level locks).

```
  SELECT blocked_locks.pid     AS blocked_pid,
         blocked_activity.usename  AS blocked_user,
         blocking_locks.pid     AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query    AS blocked_statement,
         blocking_activity.query   AS current_statement_in_blocking_process
   FROM  pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks 
        ON blocking_locks.locktype = blocked_locks.locktype
        AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
        AND blocking_locks.pid != blocked_locks.pid

    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.granted;
```

~~~sql
SELECT * FROM accounts WHERE id=1 FOR UPDATE; 这条语句阻塞了
~~~

~~~sql
Here's an alternate view of that same data that includes an idea how old the state is
# 列出所有锁

SELECT a.datname,
         l.relation::regclass,
         l.transactionid, //事务id
         l.mode, 锁的mod
         l.GRANTED,
         a.usename,  who
         a.query, 
         a.query_start,
         age(now(), a.query_start) AS "age",
         a.pid
FROM pg_stat_activity a
JOIN pg_locks l ON l.pid = a.pid
ORDER BY a.query_start;
~~~



**死锁是由外键约束引起的**

1.删除约束

修改sql

~~~sql
-- name: GetAccountForUpdate :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1
FOR NO KEY UPDATE;//这步时解决死锁的关键
~~~

避免死锁是关键：微调事务中的查询



### 十三.隔离级别

数据库事务必须满足 ACID 原子性  一致性 隔离性 持久性

----



#### Read Phenomenaa



#### 1.脏读

当一个事务读取了   其他并发事务写入的尚未提交的数据（导致 如果尚未提交的数据 最终回滚 可能导致用到错误的数据 ）

#### 2.不可重复读

当一个事务两次读取到同一记录并看到不同的值  因为第一次读取后提交的其他事务修改

#### 3.幻读

影响多行

#### 4.四种隔离级别

READ UNCOMMITMED： 可以看到其他未提交事务写入的数据

READ COMMITED：只能看到其他事务已经提交的数据

REPEATABLE READ: 

SERIALIZABLE:



#### mysql选择隔离级别

~~~sql
set sexxion transaction isolation level read commited;

select @@一种隔离级别
~~~

#### postgresql选择隔离级别 只有三个

~~~
在postgresql中 未提交和已提交是一个级别
show transaction isolation level

set transaction isolation level read uncommited

~~~

### 十四.持续集成或CI



**自动化构建和测试流程进行验证**

#### 1.Github Action





首先上传项目到github时如果出现了连接问题 就切换成ssh连接

~~~shell
git remote set-url origin git@github.com:Whuichenggong/projects.git
PS E:\projects> git pull origin main --tags
From github.com:Whuichenggong/projects
 * branch            main       -> FETCH_HEAD
~~~



创建文件

~~~shell
echo. > .github\workflows\ci.yml
~~~

安装工具

~~~
golang migrate
~~~



但是目前我看不到页面我的action



### 十五.RESTful HEEP API

#### 创建api文件夹

**account.go**

~~~go
package api

import (
	"net/http"
	db "project/simplebank/db/sqlc"

	"github.com/gin-gonic/gin"
)

type CreateAccountRequest struct {
	Owner    string `json:"owner" binding:"required"`
	Currency string `json:"currency" binding:"required,oneof= USD EUR"`
}

func (server *Server) createAccount(ctx *gin.Context) {
	var req CreateAccountRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}
	arg := db.CreateAccountParams{
		Owner:    req.Owner,
		Currency: req.Currency,
		Balance:  0,
	}
	account, err := server.store.CreateAccount(ctx, arg)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}

	ctx.JSON(http.StatusOK, account)
}

~~~



**server.go**

~~~go
package api

import (
	db "project/simplebank/db/sqlc"
	"project/simplebank/util"

	"github.com/gin-gonic/gin"
)

type Server struct {
	config util.Config
	store  db.Store
	router *gin.Engine
}

func NewServer(config util.Config, store db.Store) (*Server, error) {
	server := &Server{
		config: config,
		store: store,
	}
	router := gin.Default()

	router.POST("/accounts", server.createAccount)

	server.router = router
	return server, nil
}

func errorResponse(err error) gin.H {
	return gin.H{"error": err.Error()}
}

func (server *Server) Start(address string) error {
	return server.router.Run(address)
}

~~~

**main.go**

~~~go
package main

import (
	"context"
	"log"
	"project/simplebank/api"
	"project/simplebank/util"

	db "project/simplebank/db/sqlc"

	"github.com/jackc/pgx/v5/pgxpool"
)

func main() {
	config, err := util.LoadConfig(".")
	if err != nil {
		log.Fatal("cannot load config:", err)
	}

	connPool, err := pgxpool.New(context.Background(), config.DATABASE_URL)
	if err != nil {
		log.Fatal("cannot connect to db:", err)
	}
	//初始化数据库服务
	store := db.NewStore(connPool)
	//运行gin框架
	runGinServer(config, store)
	

	if err != nil {
		log.Fatal("cannot start server:", err)
	}

}

func runGinServer(config util.Config, store db.Store) {
	server, err := api.NewServer(config, store)
	if err != nil {
		log.Fatalf("cannot create server: %v", err)
	}
	err = server.Start(config.HTTPServerAddress)
	if err != nil {
		log.Fatalf("cannot start server: %v", err)
	}
}

~~~



**数据库重置**

~~~sql
MySQL 数据库：
使用 TRUNCATE TABLE 语句：

   TRUNCATE TABLE table_name;
   
PostgreSQL 数据库：
使用 TRUNCATE TABLE 语句：

   TRUNCATE TABLE table_name RESTART IDENTITY;
   
   TRUNCATE TABLE accounts, entries RESTART IDENTITY; 同时截断两个表
~~~



**listaccount.go**

用postman请求时：//查询参数

page_id     1

page_size   5

