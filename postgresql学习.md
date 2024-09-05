postgresql学习

---

1. **psql**:

   - PostgreSQL的交互式命令行工具，用于执行SQL查询、管理数据库和表等。

2. **CREATE DATABASE**:

   - 创建一个新的数据库。

3. **DROP DATABASE**:

   - 删除一个数据库。

4. **CREATE TABLE**:

   - 创建一个新的表。

5. **DROP TABLE**:

   - 删除一个表。

6. **ALTER TABLE**:

   - 修改现有表的结构，如添加或删除列、修改列的数据类型等。

7. **INSERT INTO**:

   - 向表中插入新的数据行。

8. **SELECT**:

   - 从表中查询数据。
   -    SELECT * FROM your_table_name;
   -  SELECT * FROM users WHERE name = '赵奎元';

9. **UPDATE**:

   - 更新表中的现有数据。

10. **DELETE**:

    - 从表中删除数据。

11. **GRANT**:

    - 授予用户或角色数据库权限。

12. **REVOKE**:

    - 撤销用户或角色的数据库权限。

13. **CREATE INDEX**:

    - 创建索引以提高查询性能。

14. **DROP INDEX**:

    - 删除索引。

15. **VACUUM**:

    - 清理数据库中的死元组，回收空间并优化数据库性能。

16. **ANALYZE**:

    - 收集关于表中数据的统计信息，帮助查询优化器更好地执行查询。

17. **EXPLAIN**:

    - 显示SQL查询的执行计划，不实际执行查询。

18. **COPY**:

    - 用于在表和文件之间复制数据。

19. **TRUNCATE**:

    - 快速删除表中的所有行。

20. **COMMENT**:

    - 添加注释到数据库对象，如表、列等。

21. **CHECKPOINT**:

    - 强制数据库进行事务日志的检查点。

22. **CREATE USER**:

    - 创建新的数据库用户。

23. **DROP USER**:

    - 删除数据库用户。

24. **CREATE ROLE**:

    - 创建新的角色。

25. **DROP ROLE**:

    - 删除角色。

26. **ALTER ROLE**:

    - 修改角色属性。

27. **CREATE EXTENSION**:

    - 安装扩展，扩展PostgreSQL的功能。

28. **DROP EXTENSION**:

    - 删除扩展。

29. **SHOW**:

    - 显示当前的配置参数值。

30. **\list**, **\l**:

    - 在psql中列出所有数据库。

31. **\connect**, **\c**:

    - 在psql中连接到特定的数据库。

32. **\dt**, **\d**:

    - 在psql中列出当前数据库的所有表或指定模式的表。

33. **\du**:

    - 在psql中列出所有用户。

34. **\df**:

    - 在psql中显示所有函数。

35. **\h**:

    - 在psql中获取关于特定SQL命令的帮助。

36. **\q**:

    - 退出psql

      **切换用户**

    -    \c database_name new_username

      **展示用户权限**

    -    SHOW CURRENT_USER;

      **SHOW root;**：

      - PostgreSQL 中没有 `SHOW root;` 这样的命令。如果你想查看当前用户的权限，应该使用 `\du` 列出所有用户，或者使用 `SELECT * FROM pg_roles;` 查询所有角色及其属性。

37.创建表

结构体是

~~~go

type User struct {
	gorm.Model
	Username     string  `gorm:"type:varchar(20);not null;unique" json:"username"`
	Password     string  `gorm:"size:255;not null" json:"password"`
	Mobile       string  `gorm:"type:varchar(11);not null;unique" json:"mobile"`
	Avatar       string  `gorm:"type:varchar(255)" json:"avatar"`
	Nickname     *string `gorm:"type:varchar(20)" json:"nickname"`
	Introduction *string `gorm:"type:varchar(255)" json:"introduction"`
	Status       uint    `gorm:"type:tinyint(1);default:1;comment:'1正常, 2禁用'" json:"status"`
	Creator      string  `gorm:"type:varchar(20);" json:"creator"`
	Roles        []*Role `gorm:"many2many:user_roles" json:"roles"`
}
~~~

创建表的sql语句

~~~sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMP WITH TIME ZONE,
    username VARCHAR(20) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    mobile VARCHAR(11) NOT NULL UNIQUE,
    avatar VARCHAR(255),
    nickname VARCHAR(20),
    introduction VARCHAR(255),
    status SMALLINT DEFAULT 1 CHECK (status IN (1, 2)),
    creator VARCHAR(20)
);
~~~

结构体

~~~go
type Menu struct {
	gorm.Model
	Name       string  `gorm:"type:varchar(50);comment:'菜单名称(英文名, 可用于国际化)'" json:"name"`
	Title      string  `gorm:"type:varchar(50);comment:'菜单标题(无法国际化时使用)'" json:"title"`
	Icon       *string `gorm:"type:varchar(50);comment:'菜单图标'" json:"icon"`
	Path       string  `gorm:"type:varchar(100);comment:'菜单访问路径'" json:"path"`
	Redirect   *string `gorm:"type:varchar(100);comment:'重定向路径'" json:"redirect"`
	Component  string  `gorm:"type:varchar(100);comment:'前端组件路径'" json:"component"`
	Sort       uint    `gorm:"type:int(3) unsigned;default:999;comment:'菜单顺序(1-999)'" json:"sort"`
	Status     uint    `gorm:"type:tinyint(1);default:1;comment:'菜单状态(正常/禁用, 默认正常)'" json:"status"`
	Hidden     uint    `gorm:"type:tinyint(1);default:2;comment:'菜单在侧边栏隐藏(1隐藏，2显示)'" json:"hidden"`
	NoCache    uint    `gorm:"type:tinyint(1);default:2;comment:'菜单是否被 <keep-alive> 缓存(1不缓存，2缓存)'" json:"noCache"`
	AlwaysShow uint    `gorm:"type:tinyint(1);default:2;comment:'忽略之前定义的规则，一直显示根路由(1忽略，2不忽略)'" json:"alwaysShow"`
	Breadcrumb uint    `gorm:"type:tinyint(1);default:1;comment:'面包屑可见性(可见/隐藏, 默认可见)'" json:"breadcrumb"`
	ActiveMenu *string `gorm:"type:varchar(100);comment:'在其它路由时，想在侧边栏高亮的路由'" json:"activeMenu"`
	ParentId   *uint   `gorm:"default:0;comment:'父菜单编号(编号为0时表示根菜单)'" json:"parentId"`
	Creator    string  `gorm:"type:varchar(20);comment:'创建人'" json:"creator"`
	Children   []*Menu `gorm:"-" json:"children"`                  // 子菜单集合
	Roles      []*Role `gorm:"many2many:role_menus;" json:"roles"` // 角色菜单多对多关系
}

~~~

psql语句

~~~sql
CREATE TABLE menus (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    title VARCHAR(50) NOT NULL,
    icon VARCHAR(50),
    path VARCHAR(100) NOT NULL,
    redirect VARCHAR(100),
    component VARCHAR(100) NOT NULL,
    sort INTEGER NOT NULL DEFAULT 999,
    status SMALLINT NOT NULL DEFAULT 1,
    hidden SMALLINT NOT NULL DEFAULT 2,
    no_cache SMALLINT NOT NULL DEFAULT 2,
    always_show SMALLINT NOT NULL DEFAULT 2,
    breadcrumb SMALLINT NOT NULL DEFAULT 1,
    active_menu VARCHAR(100),
    parent_id INTEGER NOT NULL DEFAULT 0,
    creator VARCHAR(20) NOT NULL,
    CONSTRAINT fk_parent_menu FOREIGN KEY (parent_id) REFERENCES menus(id)
);

-- 添加注释
COMMENT ON COLUMN menus.name IS '菜单名称(英文名, 可用于国际化)';
COMMENT ON COLUMN menus.title IS '菜单标题(无法国际化时使用)';
COMMENT ON COLUMN menus.icon IS '菜单图标';
COMMENT ON COLUMN menus.path IS '菜单访问路径';
COMMENT ON COLUMN menus.redirect IS '重定向路径';
COMMENT ON COLUMN menus.component IS '前端组件路径';
COMMENT ON COLUMN menus.sort IS '菜单顺序(1-999)';
COMMENT ON COLUMN menus.status IS '菜单状态(正常/禁用, 默认正常)';
COMMENT ON COLUMN menus.hidden IS '菜单在侧边栏隐藏(1隐藏，2显示)';
COMMENT ON COLUMN menus.no_cache IS '菜单是否被 <keep-alive> 缓存(1不缓存，2缓存)';
COMMENT ON COLUMN menus.always_show IS '忽略之前定义的规则，一直显示根路由(1忽略，2不忽略)';
COMMENT ON COLUMN menus.breadcrumb IS '面包屑可见性(可见/隐藏, 默认可见)';
COMMENT ON COLUMN menus.active_menu IS '在其它路由时，想在侧边栏高亮的路由';
COMMENT ON COLUMN menus.parent_id IS '父菜单编号(编号为0时表示根菜单)';
COMMENT ON COLUMN menus.creator IS '创建人';
~~~













