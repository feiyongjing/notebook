# DOM简介
Dom是指文档对象模型，即整个html网页是一个文档对象

## 7种节点类型
- Document：整个文档树的顶层节点
- DocumentType：doctype元素
- Element：网页的各种HTML元素
- Attribute：网页元素的属性，比如class='right'
- Text：元素之间或元素包含的文本内容
- Comment：注释
- DocumentFragment：文档的片段

## document获取元素节点
~~~html
<div id="test">id="test"</div>
<p>这是一个段落</p>
<div class="classTest1"> class="classTest1"</div>
<div class="classTest2"> class="classTest2"</div>
<div class="classTest2"> class="classTest2"</div>

<script>
    // getElementById根据元素属性ID获取整个元素
    var a = document.getElementById('test');
    console.log("js获取整个文档元素ID属性是test的内容，如下：")
    console.log(a)

    // getElementsByTagName根据元素名称获取整个元素
    var b = document.getElementsByTagName('p');
    console.log("js获取整个文档全部p元素的内容，是一个数组，如下：")
    console.log(b)

    // getElementsByClassName根据元素class属性获取整个元素
    var c = document.getElementsByClassName('classTest1');
    console.log("js获取整个文档全部class是classTest1全部元素的内容，是一个数组，如下：")
    console.log(c)

    // querySelector接收一个css的选择器，返回第一个选择器匹配的元素
    var d = document.querySelector('.classTest2');
    console.log("js获取整个文档元素第一个匹配css选择器'classTest2'元素中的内容，如下：")
    console.log(d)

    // querySelectorAll接收一个css的选择器，返回选择器匹配的全部元素数组
    var e = document.querySelectorAll('.classTest2');
    console.log("js获取整个文档元素匹配css选择器'.classTest2'全部元素中的内容，是一个数组，如下：")
    console.log(e)

    // parentNode方法获取当前节点的父节点元素
    // previousSibling方法获取当前节点的前一个兄弟节点元素
    // nextSibling方法获取当前节点的后一个兄弟节点元素
    // childNodes方法获取当前节点的全部子节点元素
    // firstChild方法获取当前节点的第一个子节点元素
    // lastChild方法获取当前节点的最后一个子节点元素
</script>
~~~

## dom新增元素节点
~~~html
<div class="test">hello world</div>
<script>
    var testElement = document.querySelector(".test");
    
    // createElement创建一个div元素并且带有内容，注意创建的元素dom默认是不知道放在哪个元素节点下需要进行手动设置
    var a = document.createElement("div");
    // createElement创建一个文本节点，一般是放入元素节点内容
    var b = document.createTextNode("吃饭了吗？");
    // appendChild将内容追加放入元素内容尾部或者添加一个子元素
    a.appendChild(b);

    // 也可以使用innerHTML或者innerText读取和修改元素内容（也可以用于添加和去除子元素）
    // outerHTML可以获取HTML元素的完整字符串表示
    // 它们的区别是innerHTML会将内容当成HTML来识别，innerText是直接将内容当成字符串
    testElement.innerHTML += a.outerHTML;
    console.log(testElement)
</script>
~~~

## dom修改和删除元素节点
~~~html
<div class="test">hello world</div>

<script>
    // 获取一个页面中的元素并且修改其内容
    var a = document.querySelector(".test");
    var b = document.createElement("div");
    var c = document.createTextNode("张三吃饭了没有");
    // appendChild将内容追加放入元素内容尾部或者添加一个子元素
    // 也可以使用innerHTML或者innerText读取和修改元素内容（也可以用于添加和去除子元素）
    // 它们的区别是innerHTML会将内容当成HTML来识别，innerText是直接将内容当成字符串
    b.appendChild(c);
    a.appendChild(b);
    console.log(a);

    // insertBefore方法在指定子元素的前面添加添加一个子元素，注意调用者是父元素，第一参数是新的子元素，第二参数是指定子元素
    // replaceChild方法替换指定的子元素，注意调用者是父元素，第一参数是新的子元素，第二参数是指定子元素
    // removeChild方法删除指定的子元素，注意调用者是父元素，第一参数是指定子元素
</script>
~~~

## dom元素属性的创建修改、操作CSS样式
~~~html
<div class="test">hello world</div>
<div class="test1">hello world</div>
<div class="test2">hello world</div>
<script>
    var testElement = document.querySelector(".test");
    // 创建一个div元素
    var divElement = document.createElement("div");
    // createAttribute创建一个元素属性
    var divClassAttribute = document.createAttribute("class");
    // 设置元素属性的值
    divClassAttribute.value = "哈哈";
    // setAttributeNode将元素属性放入元素
    divElement.setAttributeNode(divClassAttribute);
    // getAttributeNode获取元素的知道属性
    console.log(divElement.getAttributeNode("class"))
    // 设置元素内容
    divElement.innerHTML="哈哈"
    testElement.innerHTML += divElement.outerHTML

    // 获取一个页面中的元素并且修改其内容
    var test1Element = document.querySelector(".test1");
    // 修改id、class属性、添加class、移除class、判断是否有class
    test1Element.id = "id001";
    test1Element.className = "dom001 xxx newdom001"
    test1Element.classList.add("yyy", "hhh");
    test1Element.classList.remove("xxx", "yyy");
    console.log("判断是否有hhh这个class：" + test1Element.classList.contains("hhh"));
    // 判断是否存在newdom001这个class，如果存在就移除，不存在就添加
    test1Element.classList.toggle("newdom001");

    // js操作元素的css样式
    var test2Element = document.querySelector(".test2");
    // 方式一
    // test2Element.setAttribute("style", "width: 100px;height: 100px;border: 5px solid rgb(69, 241, 244);background-color: yellow;")
    // 方式二，推荐使用这种方式可以减少字符串拼接出错
    // test2Element.style.width = "100px";
    // test2Element.style.height = "100px";
    // test2Element.style.border = "5px solid rgb(69, 241, 244)";
    // 注意下划线变驼峰
    // test2Element.style.backgroundColor = "yellow";
    // 方式三
    test2Element.style.cssText = "width: 100px;height: 100px;border: 5px solid rgb(69, 241, 244);background-color: yellow;";
</script>
~~~

