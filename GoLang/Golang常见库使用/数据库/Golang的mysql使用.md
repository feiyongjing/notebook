# go 连接mysql使用参考
https://topgoer.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C/go%E6%93%8D%E4%BD%9Cmysql/mysql%E4%BD%BF%E7%94%A8.html

# 安装golang 的mysql包
使用第三方开源的mysql库: github.com/go-sql-driver/mysql （mysql驱动）和 github.com/jmoiron/sqlx （基于mysql驱动的封装）
~~~shell
# 在项目目录下执行安装mysql包
go get github.com/go-sql-driver/mysql 
go get github.com/jmoiron/sqlx
~~~

### mysql库进行直接连接
~~~go
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
	"log"
	"math/rand"
	"net/http"
	"time"
)

type User struct {
	Id         string
	Name       string
	Password   string
	Age        int
	Gmt_create time.Time
}

func main() {
	user1 := InsertUser(fmt.Sprint(rand.Int()), "张三", "666", 18, time.Now())
	fmt.Printf("新增的用户是：%+v\n", user1)

	user2 := UpdateUser(user1.Id, 24)
	fmt.Printf("编辑后的用户是：%+v\n", user2)

	users := getUserPages(1, 10)
	fmt.Printf("user列表如下\n")
	for _, user := range users {
		fmt.Printf("%+v\n", user)
	}

	b := DeleteUser(user2.Id)
	fmt.Printf("用户%s是否删除成功%v\n", user2.Name, b)
}

var Db *sqlx.DB

// 被引用时调用init函数，会自动的初始化
func init() {
	//database, err := sqlx.Open("数据库类型", "数据库用户名:数据库密码@tcp(ip地址:端口)/数据库名称")
	// 如果MySQL有TIMESTAMP和DATETIME类型的字段需要映射为go中的time.Time，则需要添加parseTime=True&loc=Local表示需要转换时间和确认时区
	// 注意sqlx.Open只是检查连接参数是否正确，而不是真正的建立连接
	database, err := sqlx.Open("mysql", "root:cloudgrow.cn@tcp(192.168.0.119:30306)/wz_shoe?parseTime=True&loc=Local")
	if err != nil {
		fmt.Println("open mysql failed,", err)
		return
	}

	// 重要：检查数据库连接是否真正建立
	err = database.Ping()
	if err != nil {
		log.Fatalf("数据库连接异常: %s", err)
	}

	fmt.Println("数据库连接成功")
	Db = database
}

func InsertUser(id, name, password string, age int, gmt_create time.Time) User {
	r, err := Db.Exec("insert into test(id, name, password, age, gmt_create)values(?, ?, ?, ?, ?)", id, name, password, age, gmt_create)
	if err != nil {
		panic("新增用户失败")
	}

	// 查询sql执行影响的数据行数
	row, err := r.RowsAffected()
	if err != nil || row != 1 {
		panic("新增用户失败")
	}

	return getUserById(fmt.Sprintf("%v", id))
}

func DeleteUser(id string) bool {
	res, err := Db.Exec("delete from test where id=?", id)
	if err != nil {
		panic("删除用户失败")
	}
	// 查询sql执行影响的数据行数
	row, err := res.RowsAffected()
	if err != nil || row != 1 {
		panic("删除用户失败")
	}
	return true
}

func UpdateUser(id string, age int) User {
	res, err := Db.Exec("update test set age=? where id=?", age, id)
	if err != nil {
		panic("编辑用户失败")
	}
	// 查询sql执行影响的数据行数
	row, err := res.RowsAffected()
	if err != nil || row != 1 {
		panic("编辑用户失败")
	}
	return getUserById(fmt.Sprintf("%v", id))
}

func getUserById(id string) User {
	var user []User
	err := Db.Select(&user, "select * from test where id=?", id)
	if err != nil {
		panic("根据用户ID获取用户失败")
	}
	return user[0]
}

func getUserPages(current, size int) []User {
	var user []User
	err := Db.Select(&user, "select * from test limit ?, ?", (current-1)*size, size)
	if err != nil {
		panic("分页查询用户列表失败")
	}
	return user
}

func TestTransaction() {
    // Db.Begin开启事务
	conn, err := Db.Begin()
	if err != nil {
		fmt.Println("begin failed :", err)
		return
	}

	// 执行sql
    r, err := conn.Exec("insert into person(username, sex, email)values(?, ?, ?)", "stu001", "man", "stu01@qq.com")
    if err != nil {
        fmt.Println("exec failed, ", err)
        // 执行sql出错就调用conn.Rollback进行回滚事务
        conn.Rollback()
        return
    }
	
    // 事务中的所有sql执行成功就调用conn.Commit提交事务
	conn.Commit()
}
~~~
---


# 安装golang 的GORM包
使用GORM解决数据库下划线命名（snake_case）自动映射到驼峰命名（CamelCase）的结构体属性、自动迁移、钩子（Hooks）和事务等
~~~shell
# 在项目目录下执行安装GORM包
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
~~~

### GORM库进行连接mysql的API使用
~~~go
package main

import (
    "gorm.io/gorm"
    "gorm.io/driver/mysql"
    "log"
)

type User struct {
    ID        uint   `gorm:"column:user_id"`
    Username  string `gorm:"column:user_name"`
    Email     string `gorm:"column:user_email"`
}

func main() {
    dsn := "username:password@tcp(host:port)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // 自动迁移（根据结构体调整数据库表结构）
    db.AutoMigrate(&User{})

    // 查询操作示例
    var user User
    db.First(&user, 1) // 查询id为1的用户
    log.Println("User:", user)
}
~~~





