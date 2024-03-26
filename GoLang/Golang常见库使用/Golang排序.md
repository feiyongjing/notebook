###
~~~go
package main

import (
	"fmt"
	"sort"
)

type User struct {
	Id   int64
	Name string
	Age  int32
	Node *User
}

type ById []User

func (a ById) Len() int {
	return len(a)
}

func (a ById) Swap(i, j int) {
	a[i], a[j] = a[j], a[i]
}

func (a ById) Less(i, j int) bool {
	return a[i].Id < a[j].Id
}

func main() {

	list := []User{
		{Id: 1, Name: "aa", Age: 12},
		{Id: 2, Name: "aa", Age: 11},
		{Id: 13, Name: "aa", Age: 19},
		{Id: 4, Name: "aa", Age: 9},
		{Id: 9, Name: "aa", Age: 2},
		{Id: 7, Name: "aa", Age: 5},
	}
	sort.Slice(list, func(i, j int) bool {
		return list[i].Age < list[j].Age
	})
	fmt.Println(list)

	sort.Sort(ById(list))
	fmt.Println(list)

}
~~~