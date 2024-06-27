# 默认插槽使用
### App.vue中向自定义组件标签Category写入标签内容
~~~vue
<template>
  <div id="app">
    <Category :title="美食">
      <!-- 自定义组件的标签内容存放在插槽中 -->
      <ul>
        <li v-for="item in foods" :key="item.index">{{ item }}</li>
      </ul>
    </Category>
    <Category :title="游戏">
      <!-- 自定义组件的标签内容存放在插槽中 -->
      <ul>
        <li v-for="item in games" :key="item.index">{{ item }}</li>
      </ul>
    </Category>
    <Category :title="电影">
      <!-- 自定义组件的标签内容存放在插槽中 -->
      <ul>
        <li v-for="item in films" :key="item.index">{{ item }}</li>
      </ul>
    </Category>
  </div>
</template>

<script>
import Category from "./components/Category.vue";

export default {
  name: "App",
  components: {
    Category,
  },
  data() {
    return {
      foods: ["火锅", "烧烤", "小龙虾", "牛排"],
      games: ["红色警戒", "穿越火线", "劲舞团", "超级玛丽"],
      films: ["《教父》", "《拆弹专家》", "《黑客帝国》", "《生化危机》"],
    };
  },
};
</script>

<style>
#app {
  display: flex;
  justify-content: space-around;
}
</style>
~~~

### 自定义组件Category.vue的模板设置默认插槽，以便于该自定义组件标签Category在使用时将标签内容插入插槽所在位置
~~~vue
<template>
  <div class="category">
    <h3>{{ title }}分类</h3>
    <!-- 默认插槽 -->
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "Category",
  props: ["title"],
};
</script>

<style scoped>
.category {
  background-color: aquamarine;
  width: 200px;
  height: 300px;
}
h3{
  text-align: center;
  background-color: orange;
}
</style>
~~~
---

# 具名插槽使用
### App.vue中向自定义组件标签Category写入标签内容，且设置内容标签绑定的具名插槽名称
~~~vue
<template>
  <div id="app">
    <Category :listData="foods" :title="美食">
      <!-- 自定义组件的标签内容，且设置内容标签绑定的具名插槽名称，对应的具名插槽会根据名称来获取插入的内容 -->
      <img slot="center" src="./assets/OIP-C.png" alt="" />
      <a slot="footer" href="/url">更多美食</a>
    </Category>
    <Category :title="游戏">
      <!-- 自定义组件的标签内容，且设置内容标签绑定的具名插槽名称，对应的具名插槽会根据名称来获取插入的内容 -->
      <ul slot="center">
        <li v-for="item in games" :key="item.index">{{ item }}</li>
      </ul>
      <div class="games" slot="footer">
        <a href="/url">单机游戏</a>
        <a href="/url">网络游戏</a>
      </div>
    </Category>
    <Category :title="电影">
      <!-- 自定义组件的标签内容，且设置内容标签绑定的具名插槽名称，对应的具名插槽会根据名称来获取插入的内容 -->
      <ul slot="center">
        <li v-for="item in films" :key="item.index">{{ item }}</li>
      </ul>
      <!-- 在template标签中可以使用v-slot:存放内容到具名插槽中
      (其他标签请老实的使用slot="插槽的名称")，注意插槽的名称不要添加引号 -->
      <template v-slot:footer>
        <div class="films">
          <a href="/url">经典</a>
          <a href="/url">热门</a>
          <a href="/url">推荐</a>
        </div>
        <h4 slot="footer">欢迎前来观影</h4>
      </template>
    </Category>
  </div>
</template>

<script>
import Category from "./components/Category.vue";

export default {
  name: "App",
  components: {
    Category,
  },
  data() {
    return {
      foods: ["火锅", "烧烤", "小龙虾", "牛排"],
      games: ["红色警戒", "穿越火线", "劲舞团", "超级玛丽"],
      films: ["《教父》", "《拆弹专家》", "《黑客帝国》", "《生化危机》"],
    };
  },
};
</script>

<style>
#app,
.films,
.games {
  display: flex;
  justify-content: space-around;
}

h4 {
  text-align: center;
}

img {
  width: 100%;
}
</style>
~~~

### 自定义组件Category.vue的模板设置具名插槽，具名插槽会根据名称来获取插入的内容
~~~vue
<template>
  <div class="category">
    <h3>{{ title }}分类</h3>
    <!-- 具名插槽，根据插槽的name属性来获取插入的内容 -->
    <slot name="center"></slot>
    <slot name="footer"></slot>
  </div>
</template>

<script>
export default {
  name: "Category",
  props: ["title"],
};
</script>

<style scoped>
.category {
  background-color: aquamarine;
  width: 200px;
  height: 300px;
}
h3{
  text-align: center;
  background-color: orange;
}
</style>
~~~
---

# 作用域插槽使用
### App.vue从自定义组件Category的作用域插槽中获取数据
~~~vue
<template>
  <div id="app">
    <Category :title="电影">
      <!-- 使用template标签可以通过slot-scope或者是scope从作用域插槽中获取数据 -->
      <template scope="{films}">
        <ul>
          <!-- 使用从作用域插槽中获取数据渲染列表 -->
          <li v-for="item in films" :key="item.index">{{ item }}</li>
        </ul>
        <div class="films">
          <a href="/url">经典</a>
          <a href="/url">热门</a>
          <a href="/url">推荐</a>
        </div>
        <h4>欢迎前来观影</h4>
      </template>
    </Category>

    <Category :title="电影">
      <!-- 使用template标签可以通过slot-scope或者是scope从作用域插槽中获取数据 -->
      <template scope="{films}">
        <ol>
          <!-- 使用从作用域插槽中获取数据渲染列表 -->
          <li v-for="item in films" :key="item.index">{{ item }}</li>
        </ol>
        <div class="films">
          <a href="/url">经典</a>
          <a href="/url">热门</a>
          <a href="/url">推荐</a>
        </div>
        <h4>欢迎前来观影</h4>
      </template>
    </Category>

    <Category :title="电影">
      <!-- 使用template标签可以通过slot-scope或者是scope从作用域插槽中获取数据 -->
      <template scope="{films}">
        <!-- 使用从作用域插槽中获取数据渲染列表 -->
        <h4 v-for="item in films" :key="item.index">{{ item }}</h4>
        <div class="films">
          <a href="/url">经典</a>
          <a href="/url">热门</a>
          <a href="/url">推荐</a>
        </div>
        <h4>欢迎前来观影</h4>
      </template>
    </Category>
  </div>
</template>

<script>
import Category from "./components/Category.vue";

export default {
  name: "App",
  components: {
    Category,
  },
  data() {
    return {};
  },
};
</script>

<style>
#app,
.films,
.games {
  display: flex;
  justify-content: space-around;
}

h4 {
  text-align: center;
}

img {
  width: 100%;
}
</style>
~~~

### 自定义组件Category.vue设置作用域插槽，向插槽的使用者传递一些数据，和props类似
~~~vue
<template>
  <div class="category">
    <h3>{{ title }}分类</h3>
    <!-- 作用域插槽，向插槽的使用者传递一些数据，和props类似 -->
    <slot :films="films"></slot>
  </div>
</template>

<script>
export default {
  name: "Category",
  data() {
    return {
      films: ["《教父》", "《拆弹专家》", "《黑客帝国》", "《生化危机》"],
    }
  },
  props: ["title"],
  methods: {},
};
</script>

<style scoped>
.category {
  background-color: aquamarine;
  width: 200px;
  height: 300px;
}
h3{
  text-align: center;
  background-color: orange;
}
</style>
~~~


