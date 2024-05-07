# 安装golang 的JWT包
~~~shell
# 在项目目录下执行安装JWT包
go get github.com/golang-jwt/jwt/v5
~~~

### HS256算法的签名和验签
~~~go
package main

import (
	"crypto/rand"
	"errors"
	"fmt"
	"github.com/golang-jwt/jwt/v5"
	"strings"
	"time"
)

// 签名密钥
const sign_key = "hello jwt"

// 使用 crypto/rand 库的 Read 函数填充签名密钥，确保生成的签名密钥具有高强度的随机性和不可预测性
func init() {
	sign_key := make([]byte, 32) // 生成32字节（256位）的密钥
	if _, err := rand.Read(sign_key); err != nil {
		panic(err)
	}
}

func generateTokenUsingHs256() (string, error) {
	// Claims是JWT 的声明，即JWT字符串的Payload部分，里面有一下的JWT的声明信息，例如发行人（iss）、主题（sub）等，这个可以自定义实现添加声明信息
	// 注意敏感信息和机密信息不用存放在Claims中
	claim := jwt.RegisteredClaims{
		Issuer:    "Auth_Server",                                   // 签发者
		Subject:   "Tom",                                           // 签发对象
		Audience:  jwt.ClaimStrings{"Android_APP", "IOS_APP"},      //签发受众
		ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour)),   //过期时间
		NotBefore: jwt.NewNumericDate(time.Now().Add(time.Second)), //最早使用时间
		IssuedAt:  jwt.NewNumericDate(time.Now()),                  //签发时间

	}

	// 通过jwt.NewWithClaims创建Token
	// 第一参数用于指定 JWT 的签名算法：常用的签名算法有 SigningMethodHS256、SigningMethodRS256等。这些算法分别代表不同的签名技术，如 HMAC、RSA。
	// 第二参数是Claims，即JWT 的声明信息
	// 后续的参数传递零个或多个 TokenOption 函数，它接收一个 *Token，这样就可以在创建 Token 的时候对其进行进一步的配置。
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claim)

	// 通过使用 jwt.Token 对象的 SignedString 方法，
	// 对Header、Payload进行序列化生成base64url编码
	// 然后和签名密钥一起进行签名形成Signature
	// 最终使用 . 拼接Header、Payload进行序列化生成base64url编码和Signature 生成 token 字符串
	tokenStr, err := token.SignedString([]byte(sign_key))
	return tokenStr, err
}

func parseTokenHs256(token_string string) (claims *jwt.RegisteredClaims, err error) {
	claims = &jwt.RegisteredClaims{}

	// jwt.ParseWithClaims用于解析JWT字符串，会解析出JWT声明信息，即Claims
	// 第一参数是要解析的 JWT 字符串
	// 第二参数是解析的结果，即Claims，存放JWT声明信息的对象
	// 第三参数是一个回调函数，回调函数接收jwt.Token 对象，会返回该对象用于验证 JWT 签名的签名密钥
	// 后续的参数传递零个或多个 ParserOption 类型参数。这些选项可以用来定制解析器的行为，例如设置 exp 声明为必需的参数并且没有过期，否则解析失败
	token, err := jwt.ParseWithClaims(token_string, claims, func(token *jwt.Token) (interface{}, error) {
		return []byte(sign_key), nil //返回签名密钥
	})
	if err != nil {
		return nil, err
	}

	// 验证token令牌是否有效
	if !token.Valid {
		return nil, errors.New("token 无效")
	}
	return claims, nil
}

func main() {
	token, err := generateTokenUsingHs256()
	if err != nil {
		panic(err)
	}
	fmt.Println("Token = ", token)

	time.Sleep(time.Second * 2)

	split := strings.Split(token, ".")
	
	// token中的Payload修改，会导致验证签名失败，确认信息被修改了
	payload := split[1]
	//payload = "eyJpc3MiOiJBdXRoX1NlcnZlciIsInN1YiI6Inh4eCIsImF1ZCI6WyJBbmRyb2lkX0FQUCIsIklPU19BUFAiXSwiZXhwIjoxNzE1MDgxNzUwLCJuYmYiOjE3MTUwNzgxNTEsImlhdCI6MTcxNTA3ODE1MH0"

	// token中的Signature修改签名，会导致验证签名失败，确认信息被修改了
	signature := split[2]
	//signature = signature[:len(signature)-5] + "aaaaa"

	token = split[0] + "." + payload + "." + signature
	//fmt.Println("假的Token = ", token)

	claim, err := parseTokenHs256(token)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Payload = %+v\n", *claim)
}
~~~

### RS256算法的签名和验签
~~~go
~~~





