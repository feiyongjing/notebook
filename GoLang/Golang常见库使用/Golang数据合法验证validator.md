# validator库用于对数据进行校验

## 安装
~~~shell
go get github.com/go-playground/validator/v10
~~~

# validator库使用
### 范围约束的字段类型有以下几种：
- 数值类型约束其值大小
- 字符串类型约束其长度
- 切片、数组和map，则约束其长度

# 使用validator检测结构体的合法性
~~~go
package main

import (
	"fmt"
	"github.com/go-playground/validator/v10"
	"time"
)

type User struct {
	Name      string    `validate:"ne=admin"`   // 通过在结构体字段中使用validate标记设置验证规则
	Age       int       `validate:"gte=18"`
	Sex       string    `validate:"oneof=male female"`
	RegTime   time.Time `validate:"lte"`
}

type User1 struct {
	Password string `validate:"min=10"`
}

func main() {
	// 创建validator验证器
	validate := validator.New()

	u1 := User{Name: "dj", Age: 18, Sex: "male", RegTime: time.Now().UTC()}

	// 通过调用validator验证器的Struct函数进行对传入的结构体对象
	// 根据对应结构体字段的validate标记进行验证结构体字段是否合法
	err := validate.Struct(u1)
	if err != nil {
		fmt.Println(err)
	}

	u2 := User{Name: "admin", Age: 15, Sex: "none", RegTime: time.Now().UTC().Add(1 * time.Hour)}
	err = validate.Struct(u2)
	if err != nil {
		fmt.Println(err)
	}
}
~~~

# 约束规则设置
## 对结构体字段常见的validator标记范围约束规则
- len：长度等于参数值，例如len=10
- max：值小于等于参数值，例如max=10
- min：值大于等于参数值，例如min=10
- eq：值等于参数值，注意与len不同。对于字符串，eq约束字符串本身的值，而len约束字符串长度。例如eq=10
- ne：值不等于参数值，例如ne=10
- gt：值大于参数值，例如gt=10
- gte：值大于等于参数值，例如gte=10
- lt：值小于参数值，例如lt=10
- lte：值小于等于参数值，例如lte=10
- oneof：值只能是列举出的值其中一个，这些值必须是数值或字符串，以空格分隔，如果字符串中有空格，将字符串用单引号包围，例如oneof=red green

## validator中字符串的常见约束
- contains=：包含参数子串，例如contains=email
- containsany：包含参数中任意的 UNICODE 字符，例如containsany=abcd
- containsrune：包含参数表示的 rune 字符，例如containsrune=☻
- excludes：不包含参数子串，例如excludes=email
- excludesall：不包含参数中任意的 UNICODE 字符，例如excludesall=abcd
- excludesrune：不包含参数表示的 rune 字符，excludesrune=☻
- startswith：以参数子串为前缀，例如startswith=hello
- endswith：以参数子串为后缀，例如endswith=bye

### 注意如果字段类型是time.Time，使用gt/gte/lt/lte等约束时不用指定参数值，默认与当前的 UTC 时间比较

## 跨字段约束，即与当前结构体中的其他字段比较的约束，例子如下
~~~go
type User struct {
	Name      string    `validate:"ne=admin"`
	Age       int       `validate:"gte=18"`
	Password1 string    `validate:"eqcsfield = User1.Password"`  // eqcsfield进行设置与结构体继承的指定类型下的字段相等
	Password2 string    `validate:"eqfield=Password1"`           // eqfield设置于当前结构体中的字段相等
	User1
}

type User1 struct {
	Password string `validate:"min=10"`
}
~~~

## validator中使用unqiue来指定唯一性约束，对不同类型的处理如下：
- 对于数组和切片，unique约束没有重复的元素；
- 对于map，unique约束没有重复的值；
- 对于元素类型为结构体的切片，unique约束结构体对象的某个字段不重复，通过unqiue=field指定这个字段名。
~~~go
type User struct {
  Name    string                   `validate:"min=2"`
  Age     int                      `validate:"min=18"`
  MyMap   map[string]string        `validate:"unique"`
  Hobbies []string                 `validate:"unique"`
  Friends []User                   `validate:"unique=Name"`
}
~~~

## validator中使用email限制字段必须是邮件格式
~~~go
type User struct {
  Name  string `validate:"min=2"`
  Age   int    `validate:"min=18"`
  Email string `validate:"email"`
}
~~~

## validator中一些常见的特殊检验字段设置
- -：跳过该字段，不检验
- |：使用多个约束，只需要满足其中一个，例如rgb|rgba
- required：字段必须设置，不能为默认值
- omitempty：如果字段未设置，则忽略它
~~~go
type User struct {
  Name  string            `validate:"required"`
  Age   int               `validate:"min=18 | max=35"`
  Password string         `validate:"-"`
  Remark string           `validate:"omitempty"`
}
~~~

## validator自定义约束规则
~~~go
package main

import (
	"fmt"
	"github.com/go-playground/validator/v10"
)

type RegisterForm struct {
	Name string `validate:"palindrome"` // 使用注册自定义的约束规则函数时对应的tag标记
	Age  int    `validate:"min=18"`
}

// 翻转字符串
func reverseString(s string) string {
	runes := []rune(s)
	for from, to := 0, len(runes)-1; from < to; from, to = from+1, to-1 {
		runes[from], runes[to] = runes[to], runes[from]
	}

	return string(runes)
}

// 自定义的约束规则函数
func CheckPalindrome(fl validator.FieldLevel) bool {
	value := fl.Field().String()
	return value == reverseString(value)
}

func main() {
	validate := validator.New()
	// 向validator验证器注册自定义的规则约束函数和对应的tag标记，在结构体字段中使用该标记
	validate.RegisterValidation("palindrome", CheckPalindrome)

	f1 := RegisterForm{
		Name: "djd",
		Age:  18,
	}

	// 通过调用validator验证器的Struct函数进行对传入的结构体对象
	// 根据对应结构体字段的validate标记进行验证结构体字段是否合法
	err := validate.Struct(f1)
	if err != nil {
		fmt.Println(err)
	}

	f2 := RegisterForm{
		Name: "dj",
		Age:  18,
	}
	err = validate.Struct(f2)
	if err != nil {
		fmt.Println(err)
	}
}
~~~

# validator验证失败的错误处理
### validator返回的错误实际上只有两种，一种是参数错误，一种是校验错误
- 参数错误时，返回InvalidValidationError类型
- 校验错误时返回ValidationErrors，它们都实现了error接口。而且ValidationErrors是一个错误切片，它保存了每个字段违反的每个约束信息

### 所以validator校验返回的结果只有 3 种情况
- nil：没有错误
- InvalidValidationError：输入参数错误，例如参数类型不匹配之类的问题
- ValidationErrors：字段违反validator规则约束
~~~go
package main

import (
	"fmt"
	"github.com/go-playground/validator/v10"
	"time"
)

type User struct {
	Name    string    `validate:"ne=admin"` // 通过在结构体字段中使用validate标记设置验证规则
	Age     int       `validate:"gte=18"`
	Sex     string    `validate:"oneof=male female"`
	RegTime time.Time `validate:"lte"`
}

type User1 struct {
	Password string `validate:"min=10"`
}

func main() {
	// 创建validator验证器
	validate := validator.New()

	u1 := User{Name: "dj", Age: 18, Sex: "male", RegTime: time.Now().UTC()}

	// 验证结构体字段是否合法
	err := validate.Struct(u1)
	processErr(err)

	u2 := User{Name: "admin", Age: 15, Sex: "none", RegTime: time.Now().UTC().Add(1 * time.Hour)}
	err = validate.Struct(u2)
	processErr(err)
}

func processErr(err error) {
	if err == nil {
		// 没有错误
		return
	}

	invalid, ok := err.(*validator.InvalidValidationError)
	if ok {
		// 输入的参数错误，例如类型不匹配之类的
		fmt.Println("param error:", invalid)
		return
	}

	validationErrs := err.(validator.ValidationErrors)
	for _, validationErr := range validationErrs {
		// 字段违反了validator约束规则
		fmt.Println(validationErr)
	}
}
~~~



