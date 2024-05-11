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

# gorm 的优点和缺点
- 优点：代码工整、减少代码量、解耦数据访问层与数据库方便切换数据库
- 缺点：orm内部大量使用反射影响性能、对应sql的熟练程度有一定的要求、对应ORM中设置数据实体类的关系设置有冗余性并且内部会默认使用join导致数据量大的情况下性能非常差
~~~
~~~

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
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
	"time"
)

type User struct {
	Id        string
	Name      string
	Password  string
	Age       int            `gorm:"default:18"` // gorm新增数据如果字段没有值时设置的默认值
	CreatedAt time.Time      // 创建时间（由GORM自动管理）
	UpdatedAt time.Time      // 最后一次更新时间（由GORM自动管理）
	DeletedAt gorm.DeletedAt `gorm:"index"` // 逻辑软删除（由GORM自动管理）
}

type Order struct {
	UserId     int
	FinishedAt *time.Time
}

type Result struct {
	Name  string
	Email string
}

var Db *gorm.DB

// 被引用时调用init函数，会自动的初始化
func init() {
	// dataSourceName := "数据库用户名:数据库密码@tcp(ip地址:端口)/数据库名称"
	// 如果MySQL有TIMESTAMP和DATETIME类型的字段需要映射为go中的time.Time，则需要添加parseTime=True&loc=Local表示需要转换时间和确认时区
	dataSourceName := "root:cloudgrow.cn@tcp(192.168.0.119:30306)/wz_shoe?parseTime=True&loc=Local"
	database, err := gorm.Open(mysql.Open(dataSourceName), &gorm.Config{
		NamingStrategy: schema.NamingStrategy{
			SingularTable: true, // 直接使用结构体名称表示数据库表表格名称，而不是结构体名称后添加s表示数据库表格名称
		},
	})

	if err != nil {
		fmt.Println("open mysql failed,", err)
		return
	}
	
	fmt.Println("数据库连接成功")
	Db = database
}

func main() {
	user := User{Id: "003", Name: "张三", Password: "666", Age: 18}
	_, row := InsertUser(user)
	fmt.Printf("新增的用户是否成功：%+v\n", row == 1)

	user2 := User{}
	getUserById(user.Id, &user2)
	fmt.Printf("新增的用户是：%+v\n", user2)

	user.Age = 24
	UpdateUser(user)

	user3 := User{}
	getUserById(user.Id, &user3)
	fmt.Printf("编辑后的用户是：%+v\n", user3)

	users1 := []*User{
		{Id: "004", Name: "李四", Password: "123", Age: 15},
		{Id: "005", Name: "王五", Password: "456", Age: 16},
		{Id: "006", Name: "赵六", Password: "789", Age: 17},
	}
	saveBatch(users1)

	users2 := new([]User)
	getUserPages(1, 10, users2)

	fmt.Printf("user列表如下\n")
	var userIds []string
	for _, u := range *users2 {
		fmt.Printf("%+v\n", u)
		userIds = append(userIds, u.Id)
	}

	DeleteUserByIds(userIds)
}

// 新增
func InsertUser(user User) (error, int64) {
	// 使用结构体进行新增新增一条数据
	result := Db.Create(&user)

	// 原生sql
	// Db.Exec("INSERT INTO `user` (`id`, `name`, `password`, `age`, `created_at`) VALUES (?, ?, ?, ?, ?)",
	//	user.Id, user.Name, user.Password, user.Age, time.Now())

	// 设置使用指定字段的结构体字段新增一条数据
	// INSERT INTO `user` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
	// Db.Select("Name", "Age", "CreatedAt").Create(&user)

	// 设置使用指定字段之外的结构体字段新增一条数据
	// INSERT INTO `user` (`password`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
	// Db.Omit("Name", "Age", "CreatedAt").Create(&user)

	return result.Error, result.RowsAffected
}

// 批量新增
func saveBatch(users []*User) bool {
	result := Db.Create(&users)

	// 按照数量进行分批次的新增
	// Db.CreateInBatches(users, 100)

	return result.Error != nil || result.RowsAffected != int64(len(users))
}

// 根据Ids删除
func DeleteUserByIds(userIds []string) {
	// 如果你的结构体和数据库表格包含了 gorm.DeletedAt字段（该字段也被包含在gorm.Model中），那么该模型将会自动获得逻辑软删除的能力
	// 根据Ids删除 UPDATE user SET deleted_at="2013-10-29 10:23" WHERE id in ("1", "2", "3");
	//Db.Delete(&users)
	// Db.Delete(&User{}, "10")
	// Db.Delete(&User{}, []string{"1", "2", "3"})

	// sql原生操作
	// Db.Exec("DROP TABLE users WHERE id IN ?", []string{"1", "2", "3"})

	// UPDATE user SET deleted_at="2013-10-29 10:23" WHERE age = 20;
	// Db.Where("age = ?", age).Delete(&User{})

	// DELETE FROM user WHERE name LIKE "%jinzhu%" AND id IN (1,2,3);
	// Db.Delete(&[]User{{Id: "1"}, {Id: "2"}, {Id: "3"}}, "name LIKE ?", "%jinzhu%")

	// 当执行不带任何条件的批量删除时，GORM将不会运行并返回ErrMissingWhereClause 错误
	// 如果需要请使用原生SQL，或者开启AllowGlobalUpdate 模式
	Db.Exec("DELETE FROM user WHERE Id IN ?", userIds)
	// Db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})

	// Unscoped 可以用于查询逻辑软删除的记录， SELECT * FROM user WHERE age = 20 AND deleted_at IS NOT NULL;
	// Db.Unscoped().Where("age = 20").Find(&users)

	// Unscoped 也可以用于永久删除匹配的记录
	//Db.Unscoped().Delete(&users)
}

// 编辑
func UpdateUser(user User) {
	// Save是一个组合函数，如果保存值不包含主键，它将执行 Create，否则它将执行 Update (包含所有字段，即使字段是零值)
	// INSERT INTO `user` (`name`,`age`,`birthday`,`update_at`) VALUES ("jinzhu",100,"0000-00-00 00:00:00","0000-00-00 00:00:00")
	// Db.Save(&User{Name: "jinzhu", Age: 100})

	// UPDATE user SET age=24 WHERE id=111;
	Db.Save(&user)

	// sql原生操作
	// UPDATE user SET age=24 WHERE id=111;
	//Db.Exec("UPDATE user SET age=? WHERE id= ?", user.Age, user.Id)

	// 根据条件更新，注意Save 和 Model一同使用
	// UPDATE user SET name='hello', updated_at='2013-11-17 21:34:10' WHERE age=31;
	// Db.Model(&User{}).Where("age = ?", 31).Update("name", "hello")

	// 根据条件和 model 的值进行更新，注意Save 和 Model一同使用
	// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND age=31;
	// Db.Model(&User{Id: "111"}).Where("age = ?", 31).Update("name", "hello")

	// 根据结构体更新属性，只会更新非零值的字段
	// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;
	// Db.Model(&user).Updates(User{Name: "hello", Age: 18})

	// 选择 Struct 的字段（会选中这些字段更新，如果没有设置该字段默认使用类型的零值）
	// UPDATE users SET name='new_name', age=0 WHERE id=111;
	// Db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name"})

	// 选择除指定外的所有字段（会选中这些字段更新，如果没有设置该字段默认使用类型的零值）
	// Db.Model(&user).Select("*").Omit("Age").Updates(User{Name: "jinzhu", Age: 14})
}

// 根据Id查询
func getUserById(id string, user *User) {
	//var users []User

	// 注意当使用字符串时，不要使用字符串拼接，需要使用?占位符来避免SQL注入
	// SELECT * FROM user WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";
	// First默认获取查询结果的第一条数据
	Db.First(&user, "id = ?", id)

	// sql原生操作
	//Db.Raw("SELECT * FROM user WHERE id = ?", id).Scan(&users)

	// 主键Id是数字时可以直接查询
	// SELECT * FROM user WHERE id IN (1,2,3);
	// Db.Find(&users, []int{id})
	// Db.Where([]int{1, 2, 3}).Find(&users)

	// Scan和Find一样都是将结果组装到结构体
	// Db.Table("user").Select("*").Where("id = ?", []int{1, 2, 3}).Scan(&users)

	// AND
	// SELECT * FROM user WHERE name <> "jinzhu" AND age > 20;
	//Db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)

	// OR
	// SELECT * FROM user WHERE name <> "jinzhu" OR age > 20;
	//Db.Find(&users, "name <> ? OR age > ?", "jinzhu", 20)

	// IN
	// SELECT * FROM user WHERE name IN ('jinzhu','jinzhu 2');
	//Db.Find(&users, "name IN ?", []string{"jinzhu", "jinzhu 2"})

	// NOT IN
	// SELECT * FROM user WHERE name NOT IN ('jinzhu','jinzhu 2');
	//Db.Find(&users, "name NOT IN ?", []string{"jinzhu", "jinzhu 2"})

	// LIKE
	// SELECT * FROM user WHERE name LIKE '%jin%';
	//Db.Find(&users, "name LIKE ?", "%jin%")

	// Time
	// SELECT * FROM user WHERE updated_at > '2000-01-01 00:00:00';
	//Db.Find(&users, "updated_at > ?", "2000-01-01 00:00:00")

	// BETWEEN xxx到yyy之间
	// SELECT * FROM user WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
	//Db.Find(&users, "created_at BETWEEN ? AND ?", "2000-01-01 00:00:00", "2000-01-08 00:00:00")

	// 结构体查询
	// SELECT * FROM user WHERE name = "jinzhu" AND age = 20;
	//Db.Find(&users, User{Name: "jinzhu", Age: 20})

	// 结构体查询表格指定字段
	// SELECT name, age FROM user WHERE name = "jinzhu" AND age = 20;
	// Db.Select("name", "age").Find(&users, User{Name: "jinzhu", Age: 20})

	// 结构体指定字段条件查询（如果没有设置则该字段默认使用类型的零值进行查询）
	// SELECT * FROM user WHERE name = "jinzhu" AND age = 0;
	// Db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)

	// group by 和 having
	// SELECT name, sum(age) as total FROM `user` GROUP BY `name` HAVING name = "group"
	// Db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&users)

	// distinct 过滤重复数据
	// SELECT distinct name, age FROM `user`;
	// Db.Distinct("name", "age").Find(&users)

	// order by
	// SELECT * FROM user ORDER BY age desc, name asc;
	//Db.Order("age desc, name asc").Find(&users)
	//Db.Order("age desc").Order("name asc").Find(&users)
}

// 多表join
func joinTable() {
	//results := new([]Result)
	// SELECT user.name, email.email FROM `user` LEFT JOIN email ON email.user_id = user.id AND emails.email = '"jinzhu@example.org"'
	//Db.Table("user").Select("user.name AS name, email.email AS email").
	//	Joins("LEFT JOIN email ON email.user_id = user.id AND emails.email = ?", "jinzhu@example.org").Scan(results)
	//Db.Model(&User{}).Select("user.name AS name, email.email AS email").
	//  Joins("LEFT JOIN email ON email.user_id = user.id AND emails.email = ?", "jinzhu@example.org").Scan(results)

	// Join一个临时表
	//query := Db.Table("order").Select("MAX(order.finished_at) as latest").
	//	Joins("left join user user on order.user_id = user.id").Where("user.age > ?", 18).Group("order.user_id")
	//Db.Model(&Order{}).Joins("join (?) q on order.finished_at = q.latest", query).Scan(&results)
	// SELECT `order`.`user_id`,`order`.`finished_at` FROM `order`
	// join (
	//      SELECT MAX(order.finished_at) as latest FROM `order`
	//      left join user user on order.user_id = user.id
	//      WHERE user.age > 18 GROUP BY `order`.`user_id`
	// ) q
	// on order.finished_at = q.latest
}

// 分页查询
func getUserPages(current, size int, users *[]User) {
	//var users []User
	// Limit & Offset 进行分页查询
	Db.Limit(size).Offset((current - 1) * size).Find(users)

	// 原生sql
	//Db.Exec("SELECT * FROM user Limit ?, ?", (current-1)*size, size).Find(users)
}

func TestTransaction() {
	// 开始事务
	// tx := db.Begin()
	
	// 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
	// tx.Create(...)
	
	// 遇到错误时回滚事务
	// tx.Rollback()
	
	// 否则，提交事务
	// tx.Commit()
}
~~~





