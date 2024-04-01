### 测试框架testing使用

### 被测试的代码文件，例如文件名称是cal.go
~~~go
package p1

func AddUpper(a int) int {
	res := 0
	for i := 0; i <= a; i++ {
		res += i
	}
	return res
}
~~~

### 测试代码文件
- 文件名必须是 _test结尾，例如文件名称是cal_test.go
- 测试代码文件声明的包需要与被测试代码文件声明的包保持一致，其他包正常引入运行包中的代码时不会包含测试文件代码，而当运行 go test 时则会包含测试文件代码
- 运行 go test -v 会在当前目录下的文件查找单元测试代码运行单元测试，-v 参数是显示详细信息
- 运行测试代码默认显示PASS和FAIL，PASS表示运行测试通过，FAIL表示测试用例失败
- 运行 go test -v [单元测试函数]，会在当前目录下查找指定的单元测试函数进行运行测试
- 运行 go test -v [单元测试文件 原文件]，会在对指定的单元测试文件运行测试，但是注意一定要带上被测试的原文件路径
~~~go
package p1

// 测试框架testing引入
import "testing"

// 单元测试函数必须是以Test开头并且后面的第一个字符是大写
// 原因是在运行测试时测试框架 testing 会先启动一个main函数然后扫描当前目录下的所有文件，检查是否引入了测试框架 testing，并且有满足上述条件的单元测试函数，如果满足则会运行该单元测试函数，直到所有的单元测试函数运行成功，只要有一个运行不成功都会报错
// 如果需要在打印测试日志或终止测试程序需要引入 testing ，并且给单元测试函数设置 *testing.T 类型的型参
func TestAddUpper(t *testing.T) {
	arg1 := 10
	expectedValue := 55
	res := AddUpper(arg1)

	if res != expectedValue {
        // Fatalf 函数打印错误日志并且退出程序
		t.Fatalf("AddUpper(%v)执行出错，预期值=%v，但是实际值=%v\n", arg1, expectedValue, res)
	}

    // Logf 正常打印日志
	t.Logf("AddUpper(%v)执行正确\n", arg1)
}
~~~