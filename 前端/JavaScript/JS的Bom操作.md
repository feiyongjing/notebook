# BOM简介
Bom是指浏览器对象模型，即整个浏览器窗口是一个对象

# 常见的Bom对象
- window对象表示的是整个浏览器窗口，这是一个全局对象
- Navigator对象表示的是浏览器信息，通过这个对象可以识别不同的浏览器(实际内部只有userAgent属性还有用，其他属性没有意义了)
- Location对象代表的是当前浏览器的地址栏信息，即网址输入框信息，同时可以操作这个对象进行浏览器页面跳转
- History代表的是浏览器历史记录，可以通过这个对象操作浏览器的历史记录，但是由于隐私原因这个对象无法获取到具体的历史记录
- Screen代表的屏幕信息，可以通过这个对象获取用户显示器的相关信息
- window对象下也可以获取到Navigator、Location、History、Screen这些对象

## 
~~~js
// 通过navigator.userAgent可以判断浏览器类型
console.log(navigator.userAgent)

// history.back设置浏览器页面向前跳转
history.back()

// history.back设置浏览器页面向后跳转
history.forward()

// history.go设置浏览器页面向前或者向后跳转多个页面，正数表示向前跳转，负数表示向后跳转
history.go(2)
~~~









