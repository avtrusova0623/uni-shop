# 笔记

## 准备

### 1.scss依赖

### 2. uview

- 如果根目录没有package.json文件的话，请先执行如下命令： npm init -y

- **在uni-app新建的项目中 使用CMD下载uView**  

  ```
  npm install uview-ui
  ```

- **在main.js里引入和注册uView 这两句代码需要在import Vue之后**

  ```js
  import uView from "uview-ui";
  Vue.use(uView);
  ```

- **在uni.scss文件中引入uView的全局Scss主题文件**

  ```scss
  @import 'uview-ui/theme.scss';
  ```

- **在App.vue文件中引入全局共用的scss文件**

  ```vue
  <style lang="scss">
      /* 写在第一行，style标签需要加入lang="scss" */
      @import "uview-ui/index.scss";
  </style>
  ```

- **在pages.json中设置下面代码**

  ```json
  {         // 在这里添加uview组件配置模式
      "easycom": {
          "^u-(.*)": "uview-ui/components/u-$1/u-$1.vue"
      },
      
      // 下面是项目本来有的pages 
      "pages": [
          // ......
      ]
  }
  ```

- **uniapp引入uView必需依赖scss 必须要安装此插件，在uniapp的插件市场里安装scss 如果有 则忽略此步**

- **在uni-app中正常使用uView的组件了**

### 3. uni-app扩展组件的使用(uni-ui)

#### 1. 第一种

- 在HBuilderX 新建uni-app项目的模板中，选择uni-ui模板
- 在代码区键入`u`，拉出各种内置或uni-ui的组件列表，选择其中一个，即可使用该组件

#### 2. 第二种

-  通过 uni_modules 单独安装组件
- 在uni-ui的扩展组件清单中找到所需组件，使用HX直接在项目中导入该组件
- 导入后直接使用即可，无需import和注册

### 4. HX导入插件无反应

- 需要更新最新版的HX

##  一、基础结构

###  1. 配置tabBar

- pages.json

```json
"tabBar": {
		"list": [
			{
				"pagePath": "pages/home/home",
				"text": "首页",
				"iconPath": "/static/tab_icons/home.png",
				"selectedIconPath": "/static/tab_icons/home-active.png"
			},
			{
				"pagePath": "pages/cate/cate",
				"text": "分类",
				"iconPath": "/static/tab_icons/cate.png",
				"selectedIconPath": "/static/tab_icons/cate-active.png"
			},
			{
				"pagePath": "pages/cart/cart",
				"text": "购物车",
				"iconPath": "/static/tab_icons/cart.png",
				"selectedIconPath": "/static/tab_icons/cart-active.png"
			},
			{
				"pagePath": "pages/profile/profile",
				"text": "我的",
				"iconPath": "/static/tab_icons/my.png",
				"selectedIconPath": "/static/tab_icons/my-active.png"
			}
		]
	},
```

### 2. 导航条样式

- 新版本HX中pages有自带的"navigationBarTitleText"会覆盖globalStyle中的全局样式
- 若需要名称全局一致，可以注释掉pages中的"navigationBarTitleText"

```json
"globalStyle": {
		"navigationBarTextStyle": "black",
		"navigationBarTitleText": "优购",
		"navigationBarBackgroundColor": "#C00000",
		"backgroundColor": "#FFFFFF"
	}
```

## 二、 首页

### 0、配置网络请求

1. **Fly.js**(ky用的)

   - 一个支持所有JavaScript运行环境的基于Promise的、支持请求转发、强大的http请求库

   - 使用该库发起网络请求

   - 安装使用

     ```js
     // 安装： npm install flyio
     // 在微信小程序中引入
     import Fly from 'flyio/dist/npm/wx'
     const fly = new Fly() //创建fly实例
     // 配置
     ...
     ```

2. **@escook/request-miniprogram**(hm用的)

   ```js
   // 安装： npm i @escook/request-miniprogram -S 下载网络请求包
   ```

   - 在项目的 `main.js` 入口文件中，通过如下的方式进行配置

     ```js
     import { $http } from '@escook/request-miniprogram'
     
     uni.$http = $http
     // 配置请求根路径
     $http.baseUrl = 'https://api-hmugo-web.itheima.net'
     
     // 请求拦截器
     $http.beforeRequest = function (options) {
       uni.showLoading({
         title: '数据加载中...',
       })
     }
     
     // 相应拦截器
     $http.afterRequest = function () {
       uni.hideLoading()
     }
     ```

### 1. 轮播图

#### 1. 请求轮播图的数据

实现步骤：

1. 在 data 中定义轮播图的数组
2. 在 onLoad 生命周期函数中调用获取轮播图数据的方法
3. 在 methods 中定义获取轮播图数据的方法

```js
export default {
		data() {
			return {
				// 轮播图数据，初始为空
				swiperList: []
			};
		},
		onLoad(options) {
			// 1. 获取轮播图数据
			this.getSwiperData()
		},
		methods: {
			// 1. 请求轮播图方法
			async getSwiperData() {
				// 1.1 发起请求
      			const { data: res } = await uni.$http.get('/api/public/v1/home/swiperdata')
				//   console.log(res);
      			// 1.2 请求失败
      			if (res.meta.status !== 200) {
      			  return uni.showToast({
      			    title: '数据请求失败！',
      			    duration: 1500,
      			    icon: 'none',
      			  })
      			}
      			// 1.3 请求成功，为 data 中的数据赋值
      			this.swiperList = res.message
			}
		}
	}
```
#### 2. 渲染数据

- 使用u-swiper
- 点击事件失效，暂时使用原生swiper

```vue
<template>
  <view id="home">
    <!-- 轮播图 
		list： 轮播图数据
		indicator： 是否显示面板指示器
		indicatorMode： 指示器模式
		circular： 播放到末尾后是否重新回到开头
		nterval:	滑块自动切换时间间隔（ms）
		keyName:list数组里的属性
	-->
    <u-swiper 
	 :list="swiperList" 
	 keyName="image_src" 
	 :interval="3000"
	 :loading='loading'
	 indicator 
	 indicatorMode="dot" 
	 circular>
	 </u-swiper>
  </view>
</template>
```

- 原生

  ```html
  <swiper
    :indicator-dots="true"
    :autoplay="true"
    :interval="3000"
    :duration="1000"
    :circular="true"
  >
    <swiper-item v-for="(item, index) in swiperList" :key="index">
      <view class="swiper-item">
        <image :src="item.image_src"></image>
      </view>
    </swiper-item>
  </swiper>
  <style lang="scss">
  swiper {
    height: 330rpx;
  
    .swiper-item,
    image {
      width: 100%;
      height: 100%;
    }
  }
  </style>
  ```

  

#### 3. 配置小程序分包

- **分包可以减少小程序首次启动时的加载时间**
- 在项目中，把 tabBar 相关的 4 个页面放到主包中，其它页面（例如：商品详情页、商品列表页）放到分包中
- 具体步骤如下：
  - 在项目根目录中，创建分包的根目录，命名为 `subpkg`
  - 在 `pages.json` 中，和 `pages` 节点平级的位置声明 `subPackages` 节点，用来定义分包相关的结构
  - 在 `subpkg` 目录上鼠标右键，点击 `新建页面` 选项，并填写页面的相关信息：重点选择小程序分包

#### 4. 点击轮播图跳转到商品详情页面

1. 将 `<swiper-item></swiper-item>` 节点内的 `view` 组件，改造为 `navigator` 导航组件，并动态绑定 `url 属性` 的值

   ```vue
   <swiper
      :indicator-dots="true"
      :autoplay="true"
      :interval="3000"
      :duration="1000"
      :circular="true"
    >
      <swiper-item v-for="(item, index) in swiperList" :key="index">
        <navigator
          class="swiper-item"
          :url="'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id">
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>
   ```


#### 5. 封装 uni.$showMsg() 方法

- 当数据请求失败之后，经常需要调用 `uni.showToast({ /* 配置对象 */ })` 方法来提示用户

- 可以在全局封装一个 `uni.$showMsg()` 方法，来简化 `uni.showToast()` 方法的调用

- 具体步骤如下：

  - 在 `main.js` 中，为 `uni` 对象挂载自定义的 `$showMsg()` 方法

    ```js
    // 封装消息提示
    uni.$showMsg = function (title = '数据加载失败！', duration = 1500) {
      uni.showToast({
        title,
        duration,
        icon: 'none',
      })
    }
    ```

  - 在需要提示消息的时候，直接调用 `uni.$showMsg()` 方法即可

    ```js
    async getSwiperData() {
          // 1.1 发起请求
          const { data: res } = await uni.$http.get(
            "/api/public/v1/home/swiperdata"
          );
          //   console.log(res);
          // 1.2 请求失败
          if (res.meta.status !== 200) {
            // return uni.showToast({
            //   title: "数据请求失败！",
            //   duration: 1500,
            //   icon: "none",
            // });
            return uni.$showMsg()
          }
          // 1.3 请求成功，为 data 中的数据赋值
          this.swiperList = res.message;
          uni.$showMsg("请求成功")
        }
    ```

### 2. 分类导航区域

#### 1. 获取分类导航的数据

实现思路：

1. 定义 data 数据
2. 在 onLoad 中调用获取数据的方法
3. 在 methods 中定义获取数据的方法

```js
export default {
  data() {
    return {
      // 分类导航数据
      navList: []
    };
  },
  onLoad(options) {
    // 2. 获取分类导航数据
    this.getNavList()
  },
  methods: {
    // 2. 请求分类导航数据
    async getNavList() {
      const { data: res } = await uni.$http.get('/api/public/v1/home/catitems')
      if (res.meta.status !== 200) return uni.$showMsg()
      this.navList = res.message
    }
  },
};
```

#### 2. 渲染分类导航的 UI 结构

##### 1. 结构

```vue
<!-- 分类导航 -->
<view class="nav-list">
  <view class="nav-item" v-for="(item, index) in navList" :key="index">
    <image :src="item.image_src" class="nav-img"></image>
  </view>
</view>
```

##### 2. 样式

```style
/*分类导航 */
.nav-list {
  display: flex;
  justify-content: space-around;
  margin: 15px 0;

  .nav-img {
    width: 128rpx;
    height: 140rpx;
  }
}
```

##### 3. 点击分类，跳转到分类页面

1. 为 `nav-item` 绑定点击事件处理函数

   ```vue
   <view class="nav-list">
     <view class="nav-item" v-for="(item, index) in navList" :key="index" @click="navClickHandler(item)">
       <image :src="item.image_src" class="nav-img"></image>
     </view>
   </view>
   ```

2. 定义 `navClickHandler` 事件处理函数

   ```js
    // 3. 处理分类导航点击事件
    navClickHandler(item) {
      // 判断点击的是哪个 nav
      if (item.name === '分类') {
        uni.switchTab({
          url: '/pages/cate/cate'
        })
      }
    }
   ```


### 3. 首页商品展示区

#### 1. 获取商品数据

- 步骤：

  1. 定义 data 数据
  2. 在 onLoad 中调用获取数据的方法
  3. 在 methods 中定义获取数据的方法

  ```vue
  export default {
    data() {
      return {
        // 商品展示区
        showGoodsList: [],
      };
    },
    onLoad(options) {
      // 3. 获取商品展示数据
      this.getShowGoodsList();
    },
    methods: {
      //  4. 请求商品展示数据
      async getShowGoodsList() {
        const { data: res } = await uni.$http.get(
          "/api/public/v1/home/floordata"
        );
        if (res.meta.status !== 200) return uni.$showMsg();
        this.showGoodsList = res.message;
      },
    },
  };
  ```

#### 2. 渲染结构

```vue
 <!-- 展示区 -->
  <view class="floor-list">
    <view class="floor-item" v-for="(item, i) in showGoodsList" :key="i">
      <!-- 标题 -->
      <image :src="item.floor_title.image_src" class="floor-title"></image>
      <!-- 图片区域 -->
      <view class="floor-img-box">
        <!-- 左侧大图片的盒子 -->
        <view class="left-img-box">
          <image
            :src="item.product_list[0].image_src"
            :style="{ width: item.product_list[0].image_width + 'rpx' }"
            mode="widthFix"
          ></image>
        </view>
        <!-- 右侧 4 个小图片的盒子 -->
        <view class="right-img-box">
          <view
            class="right-img-item"
            v-for="(item2, i2) in item.product_list"
            :key="i2">
            <image
              v-if="i2 !== 0"
              :src="item2.image_src"
              mode="widthFix"
              :style="{ width: item2.image_width + 'rpx' }"
            ></image>
          </view>
        </view>
      </view>
    </view>
  </view>
```

#### 3. 样式

```scss
/*商品展示 */
.floor-title {
  height: 60rpx;
  width: 100%;
  display: flex;
}
.right-img-box {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.floor-img-box {
  display: flex;
  padding-left: 10rpx;
}
```

##### 4. 点击图片跳转到商品列表页

- 新建分包商品列表页goods_list

- 双层 `forEach` 循环，处理图片跳转 URL 地址

  ```js
  //  4. 请求商品展示数据
  async getShowGoodsList() {
    const { data: res } = await uni.$http.get(
      "/api/public/v1/home/floordata"
    );
    if (res.meta.status !== 200) return uni.$showMsg();
    // 双层 forEach 循环，处理 URL 地址
    res.message.forEach((floor) => {
      floor.product_list.forEach((prod) => {
        prod.url =
          "/subpkg/goods_list/goods_list?" + prod.navigator_url.split("?")[1];
      });
    });
    this.showGoodsList = res.message;
  },
  ```

- 把图片外层的 `view` 组件，改造为 `navigator` 组件，并动态绑定 `url 属性` 的值

  ```vue
  <!-- 图片区域 -->
  <view class="floor-img-box">
    <!-- 左侧大图片的盒子 -->
    <navigator class="left-img-box" :url="item.product_list[0].url">
      <image
        :src="item.product_list[0].image_src"
        :style="{ width: item.product_list[0].image_width + 'rpx' }"
        mode="widthFix"
      ></image>
    </navigator>
    <!-- 右侧 4 个小图片的盒子 -->
    <view class="right-img-box">
      <navigator
        class="right-img-item"
        v-for="(item2, i2) in item.product_list"
        :key="i2"
        :url="item2.url"
      >
        <image
          v-if="i2 !== 0"
          :src="item2.image_src"
          mode="widthFix"
          :style="{ width: item2.image_width + 'rpx' }"
        ></image>
      </navigator>
    </view>
  </view>
  ```


## 三、 分类

### 1. 获取左侧分类数据

1. 在 data 中定义分类数据节点

2. 调用获取分类列表数据的方法

3. 定义获取分类列表数据的方法

   ```js
   export default {
     data() {
       return {
         // 分类数据
         cateList: []
       };
     },
     onLoad() {
       // 1. 获取分类数据
       this.getCateList()
     },
     methods: {
       // 1. 请求分类数据
       async getCateList() {
         const { data: res } = await uni.$http.get("/api/public/v1/categories");
         if (res.meta.status !== 200) return uni.$showMsg();
         this.cateList = res.message;
       },
     },
   };
   ```

### 2. 左侧滚动区基本结构

- 结构

```vue
<!-- 左侧的滚动视图区域 -->
    <scroll-view class="left-scroll-view" scroll-y :style="{ height: wh + 'px' }">
      <view class="left-scroll-view-item active">xxx</view>
      <view class="left-scroll-view-item">xxx</view>
      <view class="left-scroll-view-item">xxx</view>
      ............
      <view class="left-scroll-view-item">多复制一些节点，演示纵向滚动效果...</view>
    </scroll-view>
```

- 样式

```scss
/* 左侧 */
.scroll-view-container {
  display: flex;

  .left-scroll-view {
    width: 240rpx;

    .left-scroll-view-item {
      line-height: 120rpx;
      background-color: #f7f7f7;
      text-align: center;
      font-size: 24rpx;

      // 激活项的样式
      &.active {
        background-color: #ffffff;
        position: relative;

        // 渲染激活项左侧的红色指示边线
        &::before {
          content: ' ';
          display: block;
          width: 6rpx;
          height: 60rpx;
          background-color: #c00000;
          position: absolute;
          left: 0;
          top: 50%;
          transform: translateY(-50%);
        }
      }
    }
  }
}
```

- 滑动区域的高度

```js
data() {
    return {
	  // 窗口的可用高度 = 屏幕高度 - navigationBar高度 - tabBar 高度
      wh: 0,
    };
  },
  onLoad() {
    // 获取当前系统的信息
    const sysInfo = uni.getSystemInfoSync();
    // 为 wh 窗口可用高度动态赋值
    this.wh = sysInfo.windowHeight;
  },
```

### 3. 动态渲染左侧滚动区数据

- 遍历v-for不是wx-for

- 类名动态渲染

- 结构

  ```vue
  <!-- 左侧的滚动视图区域 -->
  <scroll-view class="left-scroll-view" scroll-y :style="{ height: wh + 'px' }">
    <block v-for="(leftDatas,index) in leftCateList" :key="index">
      <view :class="['left-scroll-view-item', index === active ? 'active':'']"
        @click="handleLeftClick(index)"
      >{{leftDatas.cat_name}}</view>
    </block>
  </scroll-view>
  ```

- 当前选中项高亮

  ```js
  data() {
      return {
        // 当前选中项的索引，默认让第一项被选中
        active: 0
      }
  }
  // 2. 左侧导航点击事件
  handleLeftClick(index) {
    // 把被点击项的index赋值给active
    this.active = index
  }
  ```


### 4. 获取右侧分类数据

```js
data() {
    return {
      // 右侧二级分类数据
      rightCateList: [],
    }
}
// 1. 请求分类数据
async getCateList() {
  const { data: res } = await uni.$http.get("/api/public/v1/categories");
  if (res.meta.status !== 200) return uni.$showMsg();
  console.log(res);
  this.leftCateList = res.message;
  // 获取右侧分类数据，默认显示左侧第一项children数据
  this.rightCateList = res.message[0].children
},
```

### 5. 渲染右侧滚动区数据及结构

- 结构

  ```vue
  <!-- 右侧的滚动视图区域 -->
  <scroll-view class="right-scroll-view" scroll-y :style="{height: wh + 'px'}">
    <view class="cate-lv2" v-for="(rightDatas, index2) in rightCateList" :key="index2">
      <!-- 标题 -->
      <view class="cate-lv2-title">/ {{rightDatas.cat_name}} /</view>
      <!-- 分类的列表数据 -->
      <view class="cate-lv3-list">
        <view class="cate-lv3-item" v-for="(item3, index3) in rightDatas.children:key="index3">
          <!-- 图片 -->
          <image :src="item3.cat_icon"></image>
          <!-- 文本 -->
          <text>{{item3.cat_name}}</text>
        </view>
      </view>
    </view>
  </scroll-view>
  ```

- 样式

  ```scss
  /* 右侧样式 */
  .cate-lv2-title {
    font-size: 12px;
    font-weight: bold;
    text-align: center;
    padding: 15px 0;
  }
  .cate-lv3-list {
    display: flex;
    flex-wrap: wrap;
  
    .cate-lv3-item {
      width: 33.33%;
      margin-bottom: 10px;
      display: flex;
      flex-direction: column;
      align-items: center;
  
      image {
        width: 120rpx;
        height: 120rpx;
      }
  
      text {
        font-size: 12px;
      }
    }
  }
  ```

### 6、右侧滑动区设置滚动条(滚动条失效)

#### 1. 在 data 中定义 `滚动条距离顶部的距离`

#### 2. 动态为右侧的 `<scroll-view>` 组件绑定 `scroll-top` 属性的值

#### 3. 切换左侧分类时，动态设置 `scrollTop` 的值

### 7. 点击右侧分类跳转到商品列表页面

- 结构

  ```vue
  <!-- 分类的列表数据 -->
  <view class="cate-lv3-list">
    <view class="cate-lv3-item" 
    v-for="(item3, index3) in rightDatas.children" 
    :key="index3"
    @click="goGoodsList(item3.cat_id)">
      <!-- 图片 -->
      <image :src="item3.cat_icon"></image>
      <!-- 文本 -->
      <text>{{item3.cat_name}}</text>
    </view>
  </view>
  ```

- 点击事件

  ```js
  // 3. 右侧分类点击事件
  goGoodsList(id) {
    // 跳转页面
    uni.navigateTo({ 
      url: `/subpkg/goods_list/goods_list?cat_id=${id}`
    })
  }
  ```


## 四、搜索组件

### 1. 自定义搜索组件

- searchBar
- 该组件背景颜色和圆角由props传值动态决定

##### 1. 结构

```vue
<view class="my-search-container" :style="{'background-color': bgcolor}">
  <!-- 使用 view 组件模拟 input 输入框的样式 -->
  <view class="my-search-box" :style="{'border-radius': radius + 'px'}">
    <icon type="search" size="17"></icon>
    <text class="placeholder">搜索</text>
  </view>
</view>
```

##### 2. 背景颜色和圆角

```js
props: {
  // 背景颜色
  bgcolor: {
    type: String,
    default: '#C00000'
  },
  // 圆角尺寸
  radius: {
    type: Number,
    // 单位是 px
    default: 18
  }
}
```

##### 3. 样式

```scss
.my-search-container {
//   background-color: #c00000;
  height: 50px;
  padding: 0 10px;
  display: flex;
  align-items: center;
}

.my-search-box {
  height: 36px;
  background-color: #ffffff;
//   border-radius: 15px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;

  .placeholder {
    font-size: 15px;
    margin-left: 5px;
  }
}
```

### 2. 首页-搜索-跳转

#### 1. 首页-点击-跳转

```vue
<searchBar @click.native="goSearchPage"></searchBar>
// 5. 跳转到搜索页面
goSearchPage() {
  uni.navigateTo({
    url: `/subpkg/search/search`
  })
}
```

#### 2. 首页搜索吸顶效果

```vue
<view class="search-box">
  <searchBar @click.native="goSearchPage"></searchBar>
</view>
/* 搜索 */
.search-box {
  // 设置定位效果为“吸顶”
  position: sticky;
  // 吸顶的“位置”
  top: 0;
  // 提高层级，防止被轮播图覆盖
  z-index: 999;
}
```

### 3. 分类-搜索-跳转

#### 1. 搜索-跳转

- 搜索框高度50px，需要重新计算滑动区高度

  ```js
   onLoad() {
      // 1. 获取分类数据
      this.getCateList();
      // 获取当前系统的信息
      const sysInfo = uni.getSystemInfoSync();
      // 可用高度 = 屏幕高度 - navigationBar高度 - tabBar高度 - 自定义的search组件高度
      this.wh = sysInfo.windowHeight - 50
    }
  ```

- 点击跳转

  ```vue
  <searchBar @click.native="goSearchPage"></searchBar>
  // 4. 跳转到搜索页面
      goSearchPage() {
        uni.navigateTo({
          url: `/subpkg/search/search`
        })
      }
  ```

#### 2. 优化右侧滑动区

- 右侧往下滑，点击左侧分类应该跳转到对应右侧最上面

- 给scroll-view添加scroll-top

  ```vue
  <!-- 右侧的滚动视图区域 -->
  <scroll-view class="right-scroll-view" 
  scroll-y 
  :style="{height: wh + 'px'}"
  :scroll-top="scrollTop"
  >
  </scroll-view>
  <!-- 逻辑 -->
  export default {
    components: { searchBar },
    data() {
      return {
        // 滚动条距离顶部的距离
        scrollTop: 0
        
      };
    },
    methods: {
      // ========= 事件处理 ============
      // 2. 左侧导航点击事件
      handleLeftClick(index) {
        // 把被点击项的index赋值给active
        this.active = index
        // 给右侧二级分类数据赋值
        this.rightCateList = this.leftCateList[index].children
  
         // 让 scrollTop 的值在 0 与 1 之间切换
        this.scrollTop = this.scrollTop === 0 ? 1 : 0
      },
    },
  };
  ```

## 五、搜索页面

### 1. 搜索框

- 使用HX导入uni-search-bar组件

#### 1. 结构

```vue
<view class="search-box">
  <uni-search-bar
    cancelButton="none"
  :radius="100"
  placeholder="请输入搜索内容"
  @input="valueChange"
  ></uni-search-bar>
</view>
```

#### 2. 样式

- 修改 `components -> uni-search-bar -> uni-search-bar.vue` 组件，将默认的白色搜索背景改为 `#C00000` 的红色背景

```scss
.uni-searchbar {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	position: relative;
	padding: 10px;
	// background-color: #fff;
	background-color: #c00000;
}
```

#### 3. 监听输入改变事件

```js
// 1. 输入改变事件
valueChange(inputValue) {
	console.log(inputValue);
}
```

### 2. 实现搜索框自动获取焦点

- 修改 `components -> uni-search-bar -> uni-search-bar.vue` 组件，把 data 数据中的 `show` 和 `showSync` 的值，从默认的 `false` 改为 `true` 即可

```js
data() {
	return {
		show: true,
		showSync: true,
		searchVal: ''
	}
},
```

- 使用手机扫码预览，即可在真机上查看效果

### 3. 实现搜索框的防抖处理

- 定时器

  ```js
  export default {
    data() {
      return {
  		// 定时器
  		timer: null,
  		// 搜索关键字
  		keywords: ''
  	}
    },
    methods: {
  	// 1. 输入改变事件
  	valueChange(inputValue) {
  		// 清除 timer 对应的延时器
    		clearTimeout(this.timer)
    		// 重新启动一个延时器，并把 timerId 赋值给 this.timer
  		this.timer = setTimeout(()=> {
  			// 如果 500 毫秒内，没有触发新的输入事件，则为搜索关键词赋值
  			this.keywords = inputValue
  			// console.log(inputValue)
  		}, 500)
  	}
    }
  };
  ```

### 4. 搜索建议

#### 1. 请求数据

- 根据关键词查询搜索建议列表
- 只能识别汉字，不能识别拼音

```js
export default {
  data() {
    return {
        ......
		// 搜索建议结果
		searchSuggestions: []
	}
  },
  methods: {
	// 1. 输入改变事件
	valueChange(inputValue) {
			..........
			// 2. 根据关键词，查询搜索建议列表
  			this.getSearchSuggestions()
		}, 500)
	},

	// 2. 根据搜索关键词，搜索商品建议列表
	async getSearchSuggestions() {
		// 判断关键词是否为空
  		if (this.keyword === '') {
  		  this.searchSuggestions = []
  		  return
  		}

		// 请求数据
		const res = await uni.$http.get('/api/public/v1/goods/qsearch', { 
			query: this.keywords 
		})
  		if (res.data.meta.status !== 200) return uni.$showMsg()
  		this.searchSuggestions = res.data.message
	}
  }
};
```

#### 2. 渲染数据

```vue
<!-- 搜索建议列表 -->
<view class="sugg-list">
  <view
    class="sugg-item"
    v-for="(item, index) in searchSuggestions"
    :key="index"
    @click="goDetail(item.goods_id)"
  >
    <view class="goods-name">{{index+1}}-{{item.goods_name}}</view>
    <uni-icons type="arrowright" size="16"></uni-icons>
  </view>
</view>
```

#### 3. 点击列表项-跳转商品详情页

```js
// 3. 跳转到商品详情页面
goDetail(goods_id) {
  uni.navigateTo({
    // 指定详情页面的 URL 地址，并传递 goods_id 参数
    url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
  })
}
```

### 5. 搜索历史

#### 1. 获取搜索历史数据

- 输入关键词请求数据后，把关键词存进数组historyList中，为搜索历史

- 搜索历史数值数据需要排序(倒序)、去重

- 具体步骤：

  - 定义数组存放数据

  - 搜索获取请求结果后存储关键词

  - 数组去重

  - 将搜索历史记录持久化存储到本地

  - 页面加载时从本地获取历史记录

    ```js
    export default {
      data() {
        return {
          .....
          historyList: []
        };
      },
      onLoad() {
        // 加载本地存储的搜索历史记录
        this.historyList = JSON.parse(uni.getStorageSync('keywords') || '[]')
      },
      methods: {
        .......
    
        // 2. 根据搜索关键词，搜索商品建议列表
        async getSearchSuggestions() {
          ............
          // 4. 保存搜索历史数据
          this.saveSearchHistory()
        },
    	.............
    
      // 保存搜索历史
      saveSearchHistory() {
        // this.historyList.push(this.keywords)
    
        // 数组去重
        // 先把数组转换为set对象
        let set = new Set(this.historyList)
        // 每次添加数据之前先删除以确保对象中没有该值
        set.delete(this.keywords)
        // 再重新添加
        set.add(this.keywords)
        // 再把对象转为数组
        this.historyList = Array.from(set)
    
        /**
         * uni.setStorageSync(KEY,DATA)
         * 将 data 存储在本地缓存中指定的 key 中，会覆盖掉原来该 key 对应的内容
         */
        uni.setStorageSync('keywords', JSON.stringify(this.historyList))
      }  
      },
    
    };
    ```

#### 2. 渲染历史记录数据

##### 0、数组排序

- 历史数据需要倒序排列

- 计算属性histories

  ```js
  computed: {
      // 历史记录反序
      // reverse() 方法将数组中元素的位置颠倒,该方法会改变原数组
      // 以不要直接基于原数组调用 reverse 方法，以免修改原数组中元素的顺序
      histories() {
        let arr = [...this.historyList]
        return arr.reverse()
      }
   }
  ```

##### 1.  结构

```vue
<!-- 搜索历史 -->
<view class="history-box">
  <!-- 标题区域 -->
  <view class="history-title">
    <text>搜索历史</text>
    <uni-icons type="trash" size="17"></uni-icons>
  </view>
  <!-- 列表区域 -->
  <view class="history-list">
    <uni-tag :text="item" v-for="(item, index) in histories" :key="index">uni-tag>
  </view>
</view>
```

##### 2. 样式

```scss
/*历史记录 */
.history-box {
  padding: 0 5px;

  .history-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 40px;
    font-size: 13px;
    border-bottom: 1px solid #efefef;
    margin-bottom: 2px;
  }

  .history-list {
    display: flex;
    flex-wrap: wrap;

    .uni-tag {
      margin-top: 5px;
      margin-right: 5px;
    }
  }
}
```

#### 3. 清空搜索历史记录

- 为清空的图标按钮绑定 `click` 事件

    ```js
    // 清空历史记录
      clearHistory() {
        // 清空 data 中保存的搜索历史
        this.historyList = []
        // 清空本地存储中记录的搜索历史
        uni.setStorageSync('keywords', '[]')
      }
    ```

#### 4. 点击搜索历史跳转到商品列表页面

```js
// 页面跳转，商品列表页
  goGoodsList(keywords) {
    uni.navigateTo({
      url: `/subpkg/goods_list/goods_list?query=${keywords}`
    })
  }
```

### 6. 搜索建议和搜索历史的按需展示

- 当搜索结果列表的长度`不为 0`的时候，需要展示搜索建议区域，隐藏搜索历史区域
- 当搜索结果列表的长度`等于 0`的时候，需要隐藏搜索建议区域，展示搜索历史区域

```vue
<!-- 搜索建议列表 -->
<view class="sugg-list" v-if="searchSuggestions.length !== 0">
  ..............
</view>
<!-- 搜索历史 -->
<view class="history-box" v-else>
 ..............
</view>
```

## 六、商品列表

### 1. 定义请求参数

- 根据页面跳转传递的参数，请求数据参数有两类：查询关键词、商品分类id

- 请求商品数据也需要页码值及每页显示数据条数

- 将页面跳转时**携带的参数**，转存到 `queryObj` 对象中

  ```js
  data() {
      return {
          // 请求参数对象
          queryObj: {
          // 查询关键词
          query: "",
          // 商品分类Id
          cid: "",
          // 页码值
          pagenum: 1,
          // 每页显示多少条数据
          pagesize: 10,
        },
      }
  },
  onLoad(options) {
    this.queryObj.query = options.query || ''
    this.queryObj.cid = options.cat_id || ''
  },
  ```

### 2. 获取商品列表数据

- 页面加载时

  ```js
   onLoad(options) {
      ...........
  	// 调用获取商品列表数据的方法
    	this.getGoodsList()
    },
  
    methods: {
  	async getGoodsList() {
  	  // 发起请求
        const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
  	  console.log(res);
        if (res.meta.status !== 200) return uni.$showMsg()
        // 为数据赋值
        this.goodsList = res.message.goods
        this.total = res.message.total
  	}
    }
  ```

### 3. 渲染商品列表数据

- 把列表项结构封装成自定义组件goods-list-item

#### 1. 组件goods-list-item

```vue
<template>
  <view class="goods-item">
    <!-- 商品左侧图片区域 -->
    <view class="goods-item-left">
      <image
        :src="goods.goods_small_logo || defaultPic"
        class="goods-pic"
      ></image>
    </view>
    <!-- 商品右侧信息区域 -->
    <view class="goods-item-right">
      <!-- 商品标题 -->
      <view class="goods-name">{{ goods.goods_name }}</view>
      <view class="goods-info-box">
        <!-- 商品价格 -->
        <view class="goods-price">￥{{ goods.goods_price }}</view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  name: "goods-list-item",
  // 定义 props 属性，用来接收外界传递到当前组件的数据
  props: {
    // 商品的信息对象
    goods: {
      type: Object,
      defaul: {},
    },
  },
  data() {
    return {
	  // 默认的空图片
        defaultPic: 'https://img3.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png',
	}
  },
};
</script>

<style lang="scss">
.goods-item {
    display: flex;
    padding: 10px 5px;
    border-bottom: 1px solid #f0f0f0;

    .goods-item-left {
      margin-right: 5px;

      .goods-pic {
        width: 100px;
        height: 100px;
        display: block;
      }
    }

    .goods-item-right {
      display: flex;
      flex-direction: column;
      justify-content: space-between;

      .goods-name {
        font-size: 13px;
      }

      .goods-price {
        font-size: 16px;
        color: #c00000;
      }
    }
  }
</style>
```

#### 2. 使用过滤器处理价格

- 价格保留2位小数字

```js
 filters: {
    // 把数字处理为带两位小数点的数字
    tofixed(num) {
      return Number(num).toFixed(2)
    }
  }
```

```vue
<!-- 商品价格 -->
<view class="goods-price">￥{{ goods.goods_price | tofixed }}</view>
```

#### 3. 使用组件

- 商品列表页

```vue
<view class="goods-list">
    <block v-for="(item, index) in goodsList" :key="index">
      <goods-list-item :goods="item"></goods-list-item>
    </block>
  </view>
```

#### 4. 上拉加载更多商品

##### 1. 初步实现上拉加载更多

- `ages.json` 配置文件，为 `subPackages` 分包中的 `goods_list` 页面配置上拉触底的距离

  - ```js
    "onReachBottomDistance": 150
    ```

- goods_list声明 `onReachBottom` 事件处理函数，用来监听页面的上拉触底行为

  ```js
  // 触底的事件
    onReachBottom() {
      // 让页码值自增 +1
      this.queryObj.pagenum += 1
      // 重新获取列表数据
      this.getGoodsList()
    },
  ```

- 改造 `getGoodsList` 函数，当列表数据请求成功之后，进行新旧数据的拼接处理

  ```js
  methods: {
  	async getGoodsList() {
  	  // 发起请求
        const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
  	  console.log(res);
        if (res.meta.status !== 200) return uni.$showMsg()
        // 为数据赋值
        // this.goodsList = res.message.goods
  	  // 为数据赋值：通过展开运算符的形式，进行新旧数据的拼接
    	  this.goodsList = [...this.goodsList, ...res.message.goods]
        this.total = res.message.total
  	}
    },
  ```

##### 2. 通过节流阀防止发起额外的请求

- data 中定义 `isloading` 节流阀

  ```js
  data() {
      return {
          // 是否正在请求数据
          isloading: false
      }
  }
  ```

- `getGoodsList` 方法，在请求数据前后，分别打开和关闭节流阀

  ```js
  async getGoodsList() {
  	  // ** 打开节流阀
        this.isloading = true
  	  // 发起请求
        const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
  	  // ** 关闭节流阀
        this.isloading = false
  	  ................
  	}
  ```

- `onReachBottom` 触底事件处理函数中，根据节流阀的状态，来决定是否发起请求

  ```js
  // 触底的事件
    onReachBottom() {
  	// 判断是否正在请求其它数据，如果是，则不发起新的请求
      if (this.isloading) return
      .........
    }
  ```

##### 3. 判断数据是否加载完毕

- 如果下面的公式成立，则证明没有下一页数据了

  - ```text
    当前的页码值 * 每页显示多少条数据 >= 总数条数
    pagenum * pagesize >= total
    ```

- 修改 `onReachBottom` 事件处理函数

  ```js
  // 触底的事件
    onReachBottom() {
  	// 判断是否还有下一页数据
      if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) 
      return uni.$showMsg('数据加载完毕！')
  	// 判断是否正在请求其它数据，如果是，则不发起新的请求
      if (this.isloading) return
      // 让页码值自增 +1
      this.queryObj.pagenum += 1
      // 重新获取列表数据
      this.getGoodsList()
    }
  ```

#### 5. 下拉刷新

- `pages.json` 配置文件中，为 `goods_list` 页面单独开启下拉刷新效果

  - ```js
    "enablePullDownRefresh": true,
    ```

- 监听页面的 `onPullDownRefresh` 事件处理函数

  - ```js
    // 下拉刷新的事件
    onPullDownRefresh() {
      // 1. 重置关键数据
      this.queryObj.pagenum = 1
      this.total = 0
      this.isloading = false
      this.goodsList = []
    
      // 2. 重新发起请求
      this.getGoodsList(() => uni.stopPullDownRefresh())
    }
    ```

- 修改 `getGoodsList` 函数，接收 `cb` 回调函数并按需进行调用

  - ```js
    async getGoodsList(cb) {
    	  // ** 打开节流阀
          this.isloading = true
    	  // 发起请求
          const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
    	  // ** 关闭节流阀
          this.isloading = false
    	//   console.log(res);
          if (res.meta.status !== 200) return uni.$showMsg()
    
    	  // 只要数据请求完毕，就立即按需调用 cb 回调函数，停止下拉刷新
      	  cb && cb()
    
          // 为数据赋值
          // this.goodsList = res.message.goods
    	  // 为数据赋值：通过展开运算符的形式，进行新旧数据的拼接
      	  this.goodsList = [...this.goodsList, ...res.message.goods]
          this.total = res.message.total
    	}
    ```

#### 6. 点击商品 item 项跳转到详情页面

- goods_list

```vue
<view class="goods-list">
    <block v-for="(item, index) in goodsList" :key="index">
      <goods-list-item 
	  :goods="item"
	  @click.native="goDetail(item.goods_id)"
	  ></goods-list-item>
    </block>
  </view>
```

```js
// 跳转详情页
goDetail(id) {
	uni.navigateTo({
		  url: `/subpkg/goods_detail/goods_detail?goods_id=${id}`
		})
}
```

## 七、商品详情

### 1. 获取商品详情数据

```js
export default {
  data() {
    return {
      // 商品信息
      goodsInfo: {}
    }
  },

  onLoad(options) {
    // 1. 获取商品id
    const  id = options.goods_id

    // 2. 请求商品详情数据
    this.getGoodsDetail(id)
  },

  methods: {
    // 1. 请求商品数据
    async getGoodsDetail(id) {
      const { data: res } = await uni.$http.get("/api/public/v1/goods/detail", {
       goods_id: id
      })
      if (res.meta.status !== 200) return uni.$showMsg()
      // 为 data 中的数据赋值
      this.goodsInfo = res.message
    },
  },
}
```

### 2. 渲染商品详情页

#### 1. 轮播图

- 结构

  ```vue
  <!-- 轮播图 -->
  <swiper 
  :indicator-dots="true" 
  :autoplay="true" 
  :interval="3000" 
  :duration="1000" 
  :circular="true">
    <swiper-item v-for="(item, index) in goodsInfo.pics" :key="index">
      <image :src="item.pics_big"></image>
    </swiper-item>
  </swiper>
  ```

- 样式

  ```scss
  /* 轮播图 */
  swiper {
    height: 750rpx;
  
    image {
      width: 100%;
      height: 100%;
    }
  }
  ```


#### 2. 轮播图预览效果

- 为轮播图中的 `image` 图片绑定 `click` 事件处理函数

  ```vue
  <swiper 
  :indicator-dots="true" 
  :autoplay="true" 
  :interval="3000" 
  :duration="1000" 
  :circular="true">
    <swiper-item v-for="(item, index) in goodsInfo.pics" :key="index">
      <image :src="item.pics_big" @click="preview(index)"></image>
    </swiper-item>
  </swiper>
  ```

- `preview` 事件处理函数

  ```js
  // 2. 图片预览
  preview(index) {
     // 调用 uni.previewImage() 方法预览图片
     uni.previewImage({
       // 预览时，默认显示图片的索引
       current: index,
       // 所有图片 url 地址的数组
       urls: this.goodsInfo.pics.map(x => x.pics_big)
     })
  }
  ```


#### 3. 渲染商品信息区域

##### 1. 结构

```vue
<!-- 商品信息区域 -->
<view class="goods-info-box">
  <!-- 商品价格 -->
  <view class="price">￥{{goodsInfo.goods_price}}</view>
  <!-- 信息主体区域 -->
  <view class="goods-info-body">
    <!-- 商品名称 -->
    <view class="goods-name">{{goodsInfo.goods_name}}</view>
    <!-- 收藏 -->
    <view class="favi">
      <uni-icons type="star" size="18" color="gray"></uni-icons>
      <text>收藏</text>
    </view>
  </view>
  <!-- 运费 -->
  <view class="yf">快递：免运费</view>
</view>
```

##### 2. 样式

```scss
// 商品信息区域的样式
.goods-info-box {
  padding: 10px;
  padding-right: 0;

  .price {
    color: #c00000;
    font-size: 18px;
    margin: 10px 0;
  }

  .goods-info-body {
    display: flex;
    justify-content: space-between;

    .goods-name {
      font-size: 13px;
      padding-right: 10px;
    }
    // 收藏区域
    .favi {
      width: 120px;
      font-size: 12px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      border-left: 1px solid #efefef;
      color: gray;
    }
  }

  // 运费
  .yf {
    margin: 10px 0;
    font-size: 12px;
    color: gray;
  }
}
```

#### 4. 渲染商品详情介绍

##### 1. 渲染

- 在页面结构中，使用 `rich-text` 组件，将带有 HTML 标签的内容，渲染为小程序的页面结构

- goodsInfo.goods_introduce的值是html标签内容，一堆图片

  ```vue
  <!-- 商品详情介绍信息 -->
  <rich-text :nodes="goodsInfo.goods_introduce"></rich-text>
  ```

##### 2. 解决图片底部 `空白间隙` 的问题

```js
// 1. 请求商品数据
    async getGoodsDetail(id) {
      ...............
	  // 使用字符串的 replace() 方法，为 img 标签添加行内的 style 样式，从而解决图片底部空白间隙的问题
      res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g, '<img style="display:block;" ')
     ..............
    },

```

##### 3. 解决 `.webp` 格式图片在 `ios` 设备上无法正常显示的问题

```js
// 1. 请求商品数据
    async getGoodsDetail(id) {
      ...............
      // 使用字符串的 replace() 方法，将 webp 的后缀名替换为 jpg 的后缀名
  	  res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g, '<img style="display:block;" ').replace(/webp/g, 'jpg')
	  ...............
    },
```

##### 4. 解决商品价格闪烁的问题

- 在商品详情数据请求回来之前，data 中 `goodInfo` 的值为 `{}`，因此初次渲染页面时，会导致 `商品价格、商品名称` 等闪烁的问题

- 解决方案：判断 `goodsInfo.goods_name` 属性的值是否存在，从而使用 `v-if` 指令控制页面的显示与隐藏

  ```vue
  <view v-if="goodsInfo.goods_name"></view>
  ```

#### 5. 渲染详情页底部的商品导航区域

##### 1. 渲染商品导航区域的 UI 结构

- 使用**uni-ui 提供的** [GoodsNav](https://ext.dcloud.net.cn/plugin?id=865) **组件**

1. 在 data 中，通过 `options` 和 `buttonGroup` 两个数组，来声明商品导航组件的按钮配置对象

   ```js
   // 左侧按钮组的配置对象
   options: [
     {
       icon: 'chat',
       text: '客服'
     }, {
       icon: 'cart',
       text: '购物车',
       info: 2
     }],
   // 右侧按钮组的配置对象
   buttonGroup: [{
       text: '加入购物车',
       backgroundColor: '#ff0000',
       color: '#fff'
     },
     {
       text: '立即购买',
       backgroundColor: '#ffa200',
       color: '#fff'
     }
   ]
   ```

2. 在页面中使用 `uni-goods-nav` 商品导航组件

   ```vue
   <!-- 商品导航组件 -->
   <view class="goods_nav">
     <!-- fill 控制右侧按钮的样式 -->
     <!-- options 左侧按钮的配置项 -->
     <!-- buttonGroup 右侧按钮的配置项 -->
     <!-- click 左侧按钮的点击事件处理函数 -->
     <!-- buttonClick 右侧按钮的点击事件处理函数 -->
     <uni-goods-nav
      :fill="true"
      :options="options" 
      :buttonGroup="buttonGroup" 
      @click="onClick" 
      @buttonClick="buttonClick" />
   </view>
   ```

3. 美化商品导航组件，使之固定在页面最底部

   ```scss
   /*导航栏 */
   .goods-detail-container {
     // 给页面外层的容器，添加 50px 的内padding，
     // 防止页面内容被底部的商品导航组件遮盖
     padding-bottom: 50px;
   }
   
   .goods_nav {
     // 为商品导航组件添加固定定位
     position: fixed;
     bottom: 0;
     left: 0;
     width: 100%;
   }
   ```


##### 2. 点击跳转到购物车页面

- 点击购物车

  ```js
  // 3. 左侧按钮的点击事件处理函数
  onClick(e) {
    /* 事件对象 e 中会包含当前点击的按钮相关的信息,
      根据 e.content.text 的值，来决定进一步的操作
    */
    if (e.content.text === '购物车') {
      // 切换到购物车页面
      uni.switchTab({
        url: '/pages/cart/cart'
      })
    }
  },
  ```


## 八、 加入购物车

### 1. 配置vuex

- 在项目根目录中创建 `store` 文件夹，专门用来存放 vuex 相关的模块

- 新建 `index.js` 文件

- 按照如下 4 个步骤**初始化 Store 的实例对象**

  ```js
  // index.js
  // 1. 导入 Vue 和 Vuex
  import Vue from 'vue'
  import Vuex from 'vuex'
  
  // 2. 将 Vuex 安装为 Vue 的插件
  Vue.use(Vuex)
  
  // 3. 创建 Store 的实例对象
  const store = new Vuex.Store({
    // TODO：挂载 store 模块
    modules: {},
  })
  
  // 4. 向外共享 Store 的实例对象
  export default store
  ```

- 在 `main.js` 中导入 `store` 实例对象并挂载到 Vue 的实例上

  ```js
  // 1. 导入 store 的实例对象
  import store from './store/index'
  ...........
  const app = new Vue({
      ...App,
      // 2. 将 store 挂载到 Vue 实例上
      store,
  })
  ..............
  ```

### 2、创建购物车的store模块

- 新建store/modules/cart.js

  ```js
  export default {
      // 为当前模块开启命名空间
      namespaced: true,
    
      // 模块的 state 数据
      state: () => ({
        // 购物车的数组，用来存储购物车中每个商品的信息对象
        // 每个商品的信息对象，都包含如下 6 个属性：
        // { goods_id, goods_name, goods_price, goods_count, goods_small_logo, goods_state }
        cart: [],
      }),
    
      // 模块的 mutations 方法
      mutations: {},
    
      // 模块的 getters 属性
      getters: {},
    }
  ```

- 将cart.js导入index.js

  ```js
  // 1. 导入 Vue 和 Vuex
  import Vue from 'vue'
  import Vuex from 'vuex'
  
  // 1. 导入购物车的 vuex 模块
  import moduleCart from './modules/cart'
  
  // 2. 将 Vuex 安装为 Vue 的插件
  Vue.use(Vuex)
  
  // 3. 创建 Store 的实例对象
  const store = new Vuex.Store({
    // TODO：挂载 store 模块
    modules: {
      // 2. 挂载购物车的 vuex 模块，模块内成员的访问路径被调整为 m_cart，例如：
      //  购物车模块中 cart 数组的访问路径是 m_cart/cart
      m_cart: moduleCart,
    },
  })
  
  // 4. 向外共享 Store 的实例对象
  export default store
  ```

### 3. 在商品详情页中使用 Store 中的数据

#### 1. mapState 辅助方法

- computed映射

```js
// 从 vuex 中按需导出 mapState 辅助方法
import { mapState } from 'vuex'
computed: {
    // 调用 mapState 方法，把 m_cart 模块中的 cart 数组映射到当前页面中，作为计算属性来使用
    // ...mapState('模块的名称', ['要映射的数据名称1', '要映射的数据名称2'])
    ...mapState('m_cart', ['cart']),
  },
```

#### 2. 使用数据

- 在页面渲染时，可以直接使用映射过来的数据

  ```vue
  <!-- 运费 -->
  <view class="yf">快递：免运费 -- {{cart.length}}</view>
  ```

### 4. 加入购物车功能

#### 1. 封装商品加入购物车方法

- cart.js----------mutations

  ```js
   // 将商品加入购物车
   addToCart(state, goods) {
     // 根据商品的Id，查询购物车中是否存在这件商品
     // 如果不存在，则 findResult 为 undefined；否则，为查找到的商品信息对象
     const findResult = state.cart.find((x) => x.goods_id === goods.goods_id) 
     if (!findResult) {
       // 如果购物车中没有这件商品，则直接 push
       state.cart.push(goods)
     } else {
       // 如果购物车中有这件商品，则只更新数量即可
       findResult.goods_count++
     }
   },
  ```

#### 2. `mapMutations` 辅助方法

```js
import { mapState , mapMutations} from 'vuex'
 methods: {
    // 把 m_cart 模块中的 addToCart 方法映射到当前页面使用
    ...mapMutations('m_cart', ['addToCart']),
 }
```

#### 3. 点击加入购物车

- @buttonClick="buttonClick"

  ```js
  // 4. 点击加入购物车
  buttonClick(e) {
    // 1. 判断是否点击了 加入购物车 按钮
    if (e.content.text === '加入购物车') {
  
       // 2. 当前添加的商品信息对象
       const goods = {
          goods_id: this.goodsInfo.goods_id,       // 商品的Id
          goods_name: this.goodsInfo.goods_name,   // 商品的名称
          goods_price: this.goodsInfo.goods_price, // 商品的价格
          goods_count: 1,                           // 商品的数量
          goods_small_logo: this.goodsInfo.goods_small_logo, // 商品的图片
          goods_state: true                         // 商品的勾选状态
       }
  
       // 3. 通过 this 调用映射过来的 addToCart 方法，把商品信息对象存储到购物车中
       this.addToCart(goods  
       uni.$showMsg("商品添加成功")
  
    }
  },
  ```

### 5. 动态统计购物车中商品的总数量

- cart.js--------------getters------------computed

- 定义一个 `total` 方法，用来统计购物车中商品的总数量

  ```js
  // 模块的 getters 属性
  getters: {
    // 统计购物车中商品的总数量
    total(state) {
       let c = 0
       // 循环统计商品的数量，累加到变量 c 中
       state.cart.forEach(goods => c += goods.goods_count)
       return c
    }
  },
  ```

- 映射方法

  ```js
  import { mapState , mapMutations, mapGetters} from 'vuex'
  computed: {
     ..............
      ...mapGetters('m_cart', ['total']),
    },
  ```

- 通过 `watch` 侦听器，监听计算属性 `total` 值的变化，从而**动态为购物车按钮的徽标赋值**

  ```js
  watch: {
     // 1. 监听 total 值的变化，通过第一个形参得到变化后的新值
     total(newVal) {
       // 2. 通过数组的 find() 方法，找到购物车按钮的配置对象
       const findResult = this.options.find((x) => x.text === '购物车')
  
       if (findResult) {
         // 3. 动态为购物车按钮的 info 属性赋值
         findResult.info = newVal
       }
     },
   },
  ```


### 6. 持久化存储购物车中的商品

- cart.js-------mutations----------saveToStorage:将购物车中的数据持久化存储到本地

  ```js
  // 2. 将购物车中的数据持久化存储到本地
  saveToStorage(state) {
    uni.setStorageSync('cart', JSON.stringify(state.cart))
  }
  ```

- `addToCart` 方法，在处理完商品信息后，调用 `saveToStorage` 方法

  ```js
  addToCart(state, goods) {
    ..................
    // 通过 commit 方法，调用 m_cart 命名空间下的 saveToStorage 方法
    this.commit('m_cart/saveToStorage')
  },
  ```

- 读取本地存储的购物车数据，对 cart 数组进行初始化

  ```js
  cart: JSON.parse(uni.getStorageSync('cart') || '[]')
  ```


### 7. 优化商品详情页的 total 侦听器

- 使用**普通函数的形式**定义的 watch 侦听器，**在页面首次加载后不会被调用**。因此导致了商品详情页在首次加载完毕之后，不会将商品的总数量显示到商品导航区域

- 可以使用**对象的形式**来定义 watch 侦听器

  ```js
  // 定义 total 侦听器，指向一个配置对象
  total: {
     // handler 属性用来定义侦听器的 function 处理函数
     handler(newVal) {
        const findResult = this.options.find(x => x.text === '购物车')
        if (findResult) {
           findResult.info = newVal
        }
     },
     // immediate 属性用来声明此侦听器，是否在页面初次加载完毕后立即调用
     immediate: true
  },
  ```


### 8. 动态为 tabBar 页面设置数字徽标

- **从商品详情页面导航到购物车页面之后，需要为 tabBar 中的购物车动态设置数字徽标**

  - 把 Store 中的 total 映射到 `cart.vue` 中使用、

    ```js
    <script>
    import { mapGetters } from 'vuex'
    
    export default {
      data() {
        return {}
      },
      computed: {
        ...mapGetters('m_cart', ['total']),
      },
    };
    </script>
    ```

  - 在页面刚加载时立即调用 `setBadge` 方法，为 tabBar 设置数字徽标

    ```js
    onLoad(options) {
    	// 在页面刚展示的时候，设置数字徽标
    	 this.setBadge()
      }
    ```

  - setBadge方法，通过 `uni.setTabBarBadge()` 为 tabBar 设置数字徽标

    ```js
    setBadge() {
       	     // 调用 uni.setTabBarBadge() 方法，为购物车设置右上角的徽标
       	     uni.setTabBarBadge({
       	        index: 2, // 索引
       	        text: this.total + '' // 注意：text 的值必须是字符串，不能是数字
       	     })
       	  }
    ```

### 9. 将设置 tabBar 徽标的代码抽离为 mixins

- **除了要在 cart.vue 页面中设置购物车的数字徽标**

- **还需要在其它 3 个 tabBar 页面中，为购物车设置数字徽标**

- **可以使用 Vue 提供的** [mixins](https://cn.vuejs.org/v2/guide/mixins.html) **特性**

- 新建 `mixins` 文件夹，并在 `mixins` 文件夹之下新建 `tabbar-badge.js` 文件

  ```js
  import { mapGetters } from 'vuex'
  
  // 导出一个 mixin 对象
  export default {
    computed: {
      ...mapGetters('m_cart', ['total']),
    },
    onShow() {
      // 在页面刚展示的时候，设置数字徽标
      this.setBadge()
    },
    methods: {
      setBadge() {
        // 调用 uni.setTabBarBadge() 方法，为购物车设置右上角的徽标
        uni.setTabBarBadge({
          index: 2,
          text: this.total + '', // 注意：text 的值必须是字符串，不能是数字
        })
      },
    },
  }
  ```

- 在4个tabbar页面分别导入 `@/mixins/tabbar-badge.js` 模块并进行使用

  ```js
  // 导入自己封装的 mixin 模块
  import badgeMix from '@/mixins/tabbar-badge.js'
  export default {
    ..............
    // 将 badgeMix 混入到当前的页面中进行使用
    mixins: [badgeMix],
    ...............
  }
  ```

## 九、购物车页面-商品列表区域

### 0、获取购物车数据

- 通过 `mapState` 辅助函数，将 Store 中的 `cart` 数组映射到当前页面中使用

  ```js
  // 按需导入 mapState 这个辅助函数
  import { mapState } from 'vuex'
  export default {
    computed: {
       // 将 m_cart 模块中的 cart 数组映射到当前页面中使用
       ...mapState('m_cart', ['cart']),
    },
  }
  ```

### 1. 渲染购物车商品列表

- 结构

  ```vue
  <!-- 购物车商品列表的标题区域 -->
      <view class="cart-title">
        <!-- 左侧的图标 -->
        <uni-icons type="shop" size="18"></uni-icons>
        <!-- 描述文本 -->
        <text class="cart-title-text">购物车</text>
      </view>
      <!-- 商品列表区域 -->
      <block v-for="(goods, index) in cart" :key="index">
        <goods-list-item :goods="goods"></goods-list-item>
      </block>
  ```

- 样式

  ```scss
  .cart-title {
    height: 40px;
    display: flex;
    align-items: center;
    font-size: 14px;
    padding-left: 5px;
    border-bottom: 1px solid #efefef;
    .cart-title-text {
      margin-left: 10px;
    }
  }
  ```

### 2. 勾选按钮

#### 1. 为gooods-list-item组件封装 radio 勾选状态

1. 为商品的左侧图片区域添加 `radio` 组件

```vue
<!-- 商品左侧图片区域 -->
<view class="goods-item-left">
  <radio checked color="#C00000"></radio>
  <image
    :src="goods.goods_small_logo || defaultPic"
    class="goods-pic"
  ></image>
</view>

```

2. 给类名为 `goods-item-left` 的 `view` 组件添加样式，实现 `radio` 组件和 `image` 组件的左右布局

   ```scss
   .goods-item-left {
     margin-right: 5px;
     display: flex;
     justify-content: space-between;
     align-items: center;
   
     .goods-pic {
       width: 100px;
       height: 100px;
       display: block;
     }
   }
   ```

3. 封装名称为 `showRadio` 的 `props` 属性，来控制当前组件中是否显示 radio 组件

   ```js
   // 是否展示图片左侧的 radio
   showRadio: {
     type: Boolean,
     // 如果外界没有指定 show-radio 属性的值，则默认不展示 radio 组件
     default: false,
   },
   ```

4. 使用 `v-if` 指令控制 `radio` 组件的按需展示

   ```vue
   <!-- 商品左侧图片区域 -->
   <view class="goods-item-left">
     <!-- 使用 v-if 指令控制 radio 组件的显示与隐藏 -->
     <radio checked color="#C00000" v-if="showRadio"></radio>
     <image
       :src="goods.goods_small_logo || defaultPic"
       class="goods-pic"
     ></image>
   </view>
   ```

5. 在 `cart.vue` 页面中的商品列表区域，指定 `:show-radio="true"` 属性，从而显示 radio 组件

   ```vue
    <!-- 商品列表区域 -->
    <block v-for="(goods, index) in cart" :key="index">
      <goods-list-item :goods="goods" :show-radio="true"></goods-list-item>
    </block>
   ```

6. 修改 `goods-list-item.vue` 组件，动态为 `radio` 绑定选中状态

   ```vue
   <!-- 商品左侧图片区域 -->
   <view class="goods-item-left">
     <!-- 使用 v-if 指令控制 radio 组件的显示与隐藏 -->
     <!-- 存储在购物车中的商品，包含 goods_state 属性，表示商品的勾选状态 -->
     <radio :checked="goods.goods_state" color="#C00000" v-if="showRadio"></radio>
     <image
       :src="goods.goods_small_logo || defaultPic"
       class="goods-pic"
     ></image>
   </view>
   ```


#### 2. 勾选状态改变

##### 1. radio组件点击事件

- 给radio组件绑定点击事件

  ```vue
  <radio 
     :checked="goods.goods_state" 
     color="#C00000" 
     @click="radioClickHandler"
     v-if="showRadio">
  </radio>
  ```

- 在事件处理函数中发送事件radio-change

  ```js
   radioClickHandler() {
     // 通过 this.$emit() 触发外界通过 @ 绑定的 radio-change 事件，
     // 同时把商品的 Id 和 勾选状态 作为参数传递给 radio-change 事件处理函数
     this.$emit('radio-change', {
       // 商品的 Id
       goods_id: this.goods.goods_id,
       // 商品最新的勾选状态
       goods_state: !this.goods.goods_state
     })
   }
  ```

##### 2.  监听radio-change事件

- cart.vue

  ```vue
  <goods-list-item
     :goods="goods"
      :show-radio="true"
      @radio-change="radioChangeHandler()"
    >
  </goods-list-item>
  
  ```

- 处理radio-change事件

  ```js
  // 切换勾选状态
  radioChangeHandler(e) {
    // 输出得到的数据 -> {goods_id: xx, goods_state: false}
    console.log(e);
    this.updateGoodsState(e)
  }
  ```

- updateGoodsState更新商品勾选状态

  ```js
  // cart.js
   // 3. 更新购物车中商品的勾选状态
   updateGoodsState(state, goods) {
     // 根据 goods_id 查询购物车中对应商品的信息对象
     const findResult = state.cart.find(x => x.goods_id === goods.goods_id)
   
     // 有对应的商品信息对象
     if (findResult) {
       // 更新对应商品的勾选状态
       findResult.goods_state = goods.goods_state
       // 持久化存储到本地
       this.commit('m_cart/saveToStorage')
     }
   }
  ```


### 3. 商品数量

#### 1. 商品数量显示

- 使用uni-number-box组件，HX导入

- goods-list-item组件封装box组件

  ```vue
  <view class="goods-info-box">
          <!-- 商品价格 -->
          <view class="goods-price">￥{{ goods.goods_price | tofixed }}</view>
          <!-- 商品数量 -->
          <uni-number-box :min="1"></uni-number-box>
  </view>
  <style lang="scss">
  .goods-item {
      ...........
      flex: 1;
     	  ................
        .goods-info-box {
          display: flex;
          align-items: center;
          justify-content: space-between;
        }
      
        .goods-name {
         ........
        }
  	  .............
      }
    }
  </style>
  ```

#### 2. 动态为 `NumberBox` 组件绑定商品的数量值

```vue
<!-- 商品数量 -->
<uni-number-box :min="1" :value="goods.goods_count" ></uni-number-box>
```

#### 3. 动态显示 `NumberBox` 组件

- props属性动态控制数字显示

  ```js
  props: {
      // 是否展示价格右侧的 NumberBox 组件
      showNum: {
        type: Boolean,
        // 默认不显示
        default: false,
      },
  }
  ```

- v-if控制显示与否

  ```vue
  <!-- 商品数量 -->
  <uni-number-box :min="1" :value="goods.goods_count" v-if="showNum" ></uni-number-box>
  ```

- cart.vue使用goods-list-item时传递showNum

  ```vue
  <goods-list-item
         :goods="goods"
          :show-radio="true"
          :showNum="true"
          @radio-change="radioChangeHandler()"
  ></goods-list-item>
  ```

#### 4. `NumberBox` 组件商品数量改变

##### 1. 监听change事件

- 绑定事件

```vue
<uni-number-box
         :min="1"
         :value="goods.goods_count"
         @change="numChangeHandler"
         v-if="showNum" >
</uni-number-box>
```

- 事件处理

  ```js
  // 2. NumberBox 组件的 change 事件处理函数
      numChangeHandler(val) {
        // val是改变后的value值
        // console.log(val);
        // 通过 this.$emit() 触发外界通过 @ 绑定的 num-change 事件
        this.$emit('num-change', {
          // 商品的 Id
          goods_id: this.goods.goods_id,
          // 商品的最新数量
          goods_count: +val
        })
      },
  ```

##### 2. 监听num-change事件

- 绑定监听事件

  ```vue
  <goods-list-item
         :goods="goods"
          :show-radio="true"
          :showNum="true"
          @radio-change="radioChangeHandler()"
          @num-change="numberChangeHandler"
        >
  </goods-list-item>
  ```

- 事件处理

  ```js
  // 2. 商品的数量发生了变化
      numberChangeHandler(e) {
        console.log(e)
      }
  ```

#### 5. 解决 NumberBox 数据不合法的问题

- **当用户在 NumberBox 中输入字母等非法字符之后，会导致 NumberBox 数据紊乱的问题**

- 打开uni-number-box.vue` 组件，修改 `methods` 节点中的 `_onBlur函数

- 修改完毕之后，用户输入**小数**会**被转化为整数**，用户输入**非法字符**会**被替换为默认值 1**

  ```js
  _onBlur(event) {
    // 官方的代码没有进行数值转换，用户输入的 value 值可能是非法字符：
    // let value = event.detail.value;
  
    // 将用户输入的内容转化为整数
    let value = parseInt(event.detail.value);
  
    if (!value) {
      // 如果转化之后的结果为 NaN，则给定默认值为 1
      this.inputValue = 1;
      return;
    }
  
    // 省略其它代码...
  }
  ```

#### 6. 完善 NumberBox 的 inputValue 侦听器

- **在用户每次输入内容之后，都会触发 inputValue 侦听器，从而调用 this.$emit("change", newVal) 方法。这种做法可能会把不合法的内容传递出去**

- 打开uni-number-box.vue` 组件，修改 `watch` 节点中的 `inputValue 侦听器

- 修改完毕之后，NumberBox 组件只会把**合法的、且不包含小数点的新值**传递出去

  ```js
  inputValue(newVal, oldVal) {
    // 官方提供的 if 判断条件，在用户每次输入内容时，都会调用 this.$emit("change", newVal)
    // if (+newVal !== +oldVal) {
  
    // 新旧内容不同 && 新值内容合法 && 新值中不包含小数点
    if (+newVal !== +oldVal && Number(newVal) && String(newVal).indexOf('.') === -1) {
      this.$emit("change", newVal);
    }
  }
  ```

- **事实上，没有inputValue 侦听器** 

### 4. 修改购物车中商品的数量

- cart.js ---------mutations

  ```js
  // 更新购物车中商品的数量
  updateGoodsCount(state, goods) {
    // 根据 goods_id 查询购物车中对应商品的信息对象
    const findResult = state.cart.find(x => x.goods_id === goods.goods_id)
  
    if(findResult) {
      // 更新对应商品的数量
      findResult.goods_count = goods.goods_count
      // 持久化存储到本地
      this.commit('m_cart/saveToStorage')
    }
  }
  ```

- `mapMutations` 辅助函数映射updateGoodsCount

  ```js
  methods: {
      ...mapMutations('m_cart', ['updateGoodsState','updateGoodsCount']),
  
      // 2. 商品的数量发生了变化
      numberChangeHandler(e) {
        // console.log(e)
        this.updateGoodsCount(e)
        // tabbar购物车数量更新---mixins--- setBadge()
        this.setBadge()
      }
    }
  ```

### 5. 滑动删除

#### 1. 渲染滑动删除的 UI 效果

- **滑动删除需要用到 uni-ui 的 uni-swipe-action 组件和 uni-swipe-action-item**

##### 1. 将商品列表区域的结构修改

```vue
<!-- 商品列表区域 -->
    <uni-swipe-action>
       <block v-for="(goods, index) in cart" :key="index">
         <!-- uni-swipe-action-item 可以为其子节点提供滑动操作的效果。需要通过 left/right-options 属性来指定操作按钮的位置配置信息 -->
         <uni-swipe-action-item :right-options="options"="options" @click="swipeActionClickHandler(goods)">
           <goods-list-item
            :goods="goods"
             :show-radio="true"
             :showNum="true"
             @radio-change="radioChangeHandler()"
             @num-change="numberChangeHandler"
           ></goods-list-item>
         </uni-swipe-action-item>
       </block>
    </uni-swipe-action>
```

##### 2. 定义操作按钮的配置信息

- 在 data 节点中声明 `options` 数组，用来定义操作按钮的配置信息

  ```js
  data() {
      return {
        options: [{
          text: '删除', // 显示的文本内容
          style: {
            backgroundColor: '#C00000' // 按钮的背景颜色
          }
        }]
      }
    },
  ```

##### 3. 点击事件处理函数

```js
// 3. 点击了滑动操作按钮
    swipeActionClickHandler(goods) {
      console.log(goods)
    },
```

##### 4. 美化 goods-list-item组件的样式

```scss
.goods-item {
  // 让 goods-item 项占满整个屏幕的宽度
  width: 750rpx;
  // 设置盒模型为 border-box
  box-sizing: border-box;
  .............
}
```

#### 2. 实现滑动删除的功能

- 根据商品的 Id 从购物车中移除对应的商品

- cart.js------------mutations

  ```js
  // 5. 根据 Id 从购物车中删除对应的商品信息
  removeGoodsById(state, goods_id) {
    // 调用数组的 filter 方法进行过滤
    state.cart = state.cart.filter(x => x.goods_id !== goods_id)
    // 持久化存储到本地
    this.commit('m_cart/saveToStorage')
  },
  ```

- 映射方法

  ```js
  methods: {
    ...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount', 'removeGoodsById']),
    ............
    // 点击了滑动操作按钮
    swipeActionClickHandler(goods) {
        // console.log(goods)
        this.removeGoodsById(goods.goods_id)
        // tabbar购物车数量更新---mixins--- setBadge()
        this.setBadge()
      },
  ```


## 十、购物车页面-收获地址区域

### 1. 创建收获地址组件

#### 1. 结构

```vue
<view>
    <!-- 选择收货地址的盒子 -->
    <view class="address-choose-box" >
      <button type="primary" size="mini" class="btnChooseAddress">
        请选择收货地址+
      </button>
    </view>

    <!-- 渲染收货信息的盒子 -->
    <view class="address-info-box" >
      <view class="row1">
        <view class="row1-left">
          <view class="username">收货人：<text>escook</text></view>
        </view>
        <view class="row1-right">
          <view class="phone">电话：<text>138XXXX5555</text></view>
          <uni-icons type="arrowright" size="16"></uni-icons>
        </view>
      </view>
      <view class="row2">
        <view class="row2-left">收货地址：</view>
        <view class="row2-right"
          >河北省邯郸市肥乡区xxx 河北省邯郸市肥乡区xxx 河北省邯郸市肥乡区xxx
          河北省邯郸市肥乡区xxx
        </view>
      </view>
    </view>

    <!-- 底部的边框线 -->
    <image src="/static/cart_border@2x.png" class="address-border"></image>
  </view>
```

#### 2. 样式

```scss
/* 底部边框线的样式 */
.address-border {
  display: block;
  width: 100%;
  height: 5px;
}

/* 选择收货地址的盒子 */
.address-choose-box {
  height: 90px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 渲染收货信息的盒子 */
.address-info-box {
  font-size: 12px;
  height: 90px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 0 5px;

  /* 第一行 */
  .row1 {
    display: flex;
    justify-content: space-between;

    .row1-right {
      display: flex;
      align-items: center;

      .phone {
        margin-right: 5px;
      }
    }
  }

  /* 第二行 */
  .row2 {
    display: flex;
    align-items: center;
    margin-top: 10px;

    .row2-left {
      white-space: nowrap;
    }
  }
}
```

#### 3. 组件按需展示

- 在 data 中定义收货地址的信息对象

  ```js
  data() {
      return {
          address: {}
      }
  }
  ```

- 使用 `v-if` 和 `v-else` 实现按需展示

  ```vue
  <!-- 选择收货地址的盒子 -->
  <view class="address-choose-box" v-if="JSON.stringify(address) === '{}'">
    <button type="primary" size="mini" class="btnChooseAddress">请选择收货地址+</button>
  </view>
  
  <!-- 渲染收货信息的盒子 -->
  <view class="address-info-box" v-else>
    <!-- 省略其它代码 -->
  </view>
  ```

### 2. 实现选择收货地址的功能

#### 1. 为 `请选择收货地址+` 的 `button` 按钮绑定点击事件处理函数

```vue
<button
	   type="primary"
	    size="mini"
		 class="btnChooseAddress"
		 @click="chooseAddress"
		 >
        请选择收货地址+
</button>
```

```js
// 1. 选择收货地址
    async chooseAddress() {
      // 1. 调用小程序提供的 chooseAddress() 方法，即可使用选择收货地址的功能
      //    返回值是一个数组：第 1 项为错误对象；第 2 项为成功之后的收货地址对象
      const [err, succ] = await uni.chooseAddress().catch(err => err)
  
      // 2. 用户成功的选择了收货地址
      if (err === null && succ.errMsg === 'chooseAddress:ok') {
        // 为 data 里面的收货地址对象赋值
        this.address = succ
      }
    }
```

#### 2. 定义**收货详细地址**的计算属性

```js
// 收货详细地址的计算属性
    addstr() {
      if (!this.address.provinceName) return ''
  
      // 拼接 省，市，区，详细地址 的字符串并返回给用户
      return this.address.provinceName + this.address.cityName + this.address.countyName + this.address.detailInfo
    }
```

#### 3. 渲染收货地址区域的数据

  ```vue
  <!-- 渲染收货信息的盒子 -->
      <view class="address-info-box" v-else>
        <view class="row1">
          <view class="row1-left">
            <view class="username">收货人：<text>{{address.userName}}</text></view>
          </view>
          <view class="row1-right">
            <view class="phone">电话：<text>{{address.telNumber}}</text></view>
            <uni-icons type="arrowright" size="16"></uni-icons>
          </view>
        </view>
        <view class="row2">
          <view class="row2-left">收货地址：</view>
          <view class="row2-right">{{addstr}}</view>
        </view>
      </view>
  ```

### 3. 将 address 信息存储到 vuex 中

#### 1. 创建user的vuex

- store/modules/user.js

  ```js
  export default {
      // 开启命名空间
      namespaced: true,
    
      // state 数据
      state: () => ({
        // 收货地址
        address: {},
      }),
    
      // 方法
      mutations: {
        // 更新收货地址
        updateAddress(state, address) {
          state.address = address
        },
      },
    
      // 数据包装器
      getters: {},
    }
  ```

- 引入index.js

  ```js
  // 1. 导入 Vue 和 Vuex
  import Vue from 'vue'
  import Vuex from 'vuex'
  
  // 1. 导入购物车的 vuex 模块
  import moduleCart from './modules/cart'
  import moduleUser from './modules/user'
  
  // 2. 将 Vuex 安装为 Vue 的插件
  Vue.use(Vuex)
  
  // 3. 创建 Store 的实例对象
  const store = new Vuex.Store({
    // TODO：挂载 store 模块
    modules: {
      // 2. 挂载购物车的 vuex 模块，模块内成员的访问路径被调整为 m_cart，例如：
      //  购物车模块中 cart 数组的访问路径是 m_cart/cart
      m_cart: moduleCart,
      m_user: moduleUser
    },
  })
  
  // 4. 向外共享 Store 的实例对象
  export default store
  ```

#### 2. 改造 `address.vue` 组件中的代码

- 使用 **vuex 提供的 addres替代 **data 中定义的本地 address 对象**

  ```js
  // 3 . 按需导入 mapState 和 mapMutations 这两个辅助函数
  import { mapState, mapMutations } from 'vuex'
  
  export default {
    name: "address",
    data() {
      return {
  	  // 收货地址
        //   address: {},
  	}
    },
  
    computed: {
      // 3.2 把 m_user 模块中的 address 对象映射当前组件中使用，代替 data 中 address 对象
      ...mapState('m_user', ['address']),
  
      ..............
    },
  
    methods: {
  
  	// 3.1 把 m_user 模块中的 updateAddress 函数映射到当前组件
      ...mapMutations('m_user', ['updateAddress']),
  
  
  	// 1. 选择收货地址
      async chooseAddress() {
        ............
        // 2. 用户成功的选择了收货地址
        if (err === null && succ.errMsg === 'chooseAddress:ok') {
          // 为 data 里面的收货地址对象赋值
          // this.address = succ
  
  		// 3.3 调用 Store 中提供的 updateAddress 方法，将 address 保存到 Store 里面
          this.updateAddress(succ)
        }
      }
    }
  }
  ```

### 4. 将 Store 中的 address 持久化存储到本地

```js
// user.js
export default {
    // 开启命名空间
    namespaced: true,
  
    // state 数据
    state: () => ({
      // 2. 读取本地的收货地址数据，初始化 address 对象
      address: JSON.parse(uni.getStorageSync('address') || '{}'),
    }),
  
    // 方法
    mutations: {
      // 1. 更新收货地址
      updateAddress(state, address) {
        state.address = address

        // 4. 通过 this.commit() 方法，调用 m_user 模块下的 saveAddressToStorage 方法将 address 对象持久化存储到本地
        this.commit('m_user/saveAddressToStorage')
      },

      // 3. 定义将 address 持久化存储到本地 mutations 方法
      saveAddressToStorage(state) {
          uni.setStorageSync('address', JSON.stringify(state.address))
      },
    },
  
    // 数据包装器
    getters: {},
  }
```

### 5. 将 addstr 抽离为 getters

- **为了提高代码的复用性，可以把收货的详细地址抽离为 getters，方便在多个页面和组件之间实现复用**

- address.vue` 组件中的 `addstr` 计算属性的代码抽离为user.js中 getters

  ```js
   getters: {
         // 收货详细地址的计算属性
         addstr(state) {
           if (!state.address.provinceName) return ''
       
           // 拼接 省，市，区，详细地址 的字符串并返回给用户
           return state.address.provinceName + state.address.cityName + state.address.countyName + state.address.detailInfo
         }
      },
  ```

- 通过 `mapGetters` 辅助函数，将 `m_user` 模块中的 `addstr` 映射到address组件中使用

  ```js
  import { mapState, mapMutations,  mapGetters } from 'vuex'
  computed: {
      // 3.2 把 m_user 模块中的 address 对象映射当前组件中使用，代替 data 中 address 对象
      ...mapState('m_user', ['address']),
  
      // 将 m_user 模块中的 addstr 映射到当前组件中使用
      ...mapGetters('m_user', ['addstr']),
   },
  ```

### 6. 重新选择收货地址

- 为 class 类名为 `address-info-box` 的盒子绑定 `click` 事件处理函数:chooseAddress

  ```vue
  <!-- 渲染收货信息的盒子 -->
  <view class="address-info-box" v-else @click="chooseAddress">
    <!-- 省略其它代码 -->
  </view>
  ```

### 7. 解决收货地址授权失败的问题

- **如果在选择收货地址的时候，用户点击了****取消授权****，则需要进行****特殊的处理****，否则****用户将无法再次选择收货地址**

- `chooseAddress` 方法

  ```js
  // 选择收货地址
  async chooseAddress() {
    // 1. 调用小程序提供的 chooseAddress() 方法，即可使用选择收货地址的功能
    //    返回值是一个数组：第1项为错误对象；第2项为成功之后的收货地址对象
    const [err, succ] = await uni.chooseAddress().catch(err => err)
  
    // 2. 用户成功的选择了收货地址
    if (succ && succ.errMsg === 'chooseAddress:ok') {
      // 更新 vuex 中的收货地址
      this.updateAddress(succ)
    }
  
    // 3. 用户没有授权
    if (err && err.errMsg === 'chooseAddress:fail auth deny') {
      this.reAuth() // 调用 this.reAuth() 方法，向用户重新发起授权申请
    }
  }
  ```

- reAuth()

  ```js
  // 调用此方法，重新发起收货地址的授权
  async reAuth() {
    // 3.1 提示用户对地址进行授权
    const [err2, confirmResult] = await uni.showModal({
      content: '检测到您没打开地址权限，是否去设置打开？',
      confirmText: "确认",
      cancelText: "取消",
    })
  
    // 3.2 如果弹框异常，则直接退出
    if (err2) return
  
    // 3.3 如果用户点击了 “取消” 按钮，则提示用户 “您取消了地址授权！”
    if (confirmResult.cancel) return uni.$showMsg('您取消了地址授权！')
  
    // 3.4 如果用户点击了 “确认” 按钮，则调用 uni.openSetting() 方法进入授权页面，让用户重新进行授权
    if (confirmResult.confirm) return uni.openSetting({
      // 3.4.1 授权结束，需要对授权的结果做进一步判断
      success: (settingResult) => {
        // 3.4.2 地址授权的值等于 true，提示用户 “授权成功”
        if (settingResult.authSetting['scope.address']) return uni.$showMsg('授权成功！请选择地址')
        // 3.4.3 地址授权的值等于 false，提示用户 “您取消了地址授权”
        if (!settingResult.authSetting['scope.address']) return uni.$showMsg('您取消了地址授权！')
      }
    })
  }
  ```

### 8. 解决 iPhone 真机上无法重新授权的问题

- **在 iPhone 设备上，当用户取消授权之后，再次点击选择收货地址按钮的时候，无法弹出授权的提示框**

  1. 导致问题的原因 - 用户取消授权后，再次点击 “选择收货地址” 按钮的时候：

  - 在**模拟器**和**安卓真机**上，错误消息 `err.errMsg` 的值为 `chooseAddress:fail auth deny`
  - 在 **iPhone 真机**上，错误消息 `err.errMsg` 的值为 `chooseAddress:fail authorize no response`

  2. 解决问题的方案 - 修改 `chooseAddress` 方法中的代码，进一步完善用户没有授权时的 `if` 判断条件即可

     ```js
     // 4. 用户没有授权
       	  if (err && (err.errMsg === 'chooseAddress:fail auth deny' || err.errMsg === 'chooseAddress:fail authorize no response')) {
             this.reAuth()
           }
     ```

## 十一、购物车页面-结算

### 1. 创建组件settle

```vue
<template>
  <!-- 最外层的容器 -->
  <view class="my-settle-container">
    结算组件
  </view>


</template>

<script>
export default {
  name: "settle",
  data() {
    return {}
  },
}
</script>

<style lang="scss">
.my-settle-container {
  /* 底部固定定位 */
  position: fixed;
  bottom: 0;
  left: 0;
  /* 设置宽高和背景色 */
  width: 100%;
  height: 50px;
  background-color: cyan;
}
</style>
```

### 2. 使用组件

- cart.vue

```vue
 <!-- 结算区域 -->
<Settle></Settle>

<style>
.cart-container {
  padding-bottom: 50px;
}
</style>
```

### 3. 渲染结算区域的结构和样式

```vue
<template>
  <!-- 最外层的容器 -->
  <view class="my-settle-container">
    <!-- 全选区域 -->
    <label class="radio">
      <radio color="#C00000" :checked="true" /><text>全选</text>
    </label>
  
    <!-- 合计区域 -->
    <view class="amount-box">
      合计:<text class="amount">￥1234.00</text>
    </view>
  
    <!-- 结算按钮 -->
    <view class="btn-settle">结算(0)</view>
  </view>


</template>

<script>
export default {
  name: "settle",
  data() {
    return {}
  },
}
</script>

<style lang="scss">
.my-settle-container {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 50px;
  // 将背景色从 cyan 改为 white
  background-color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 5px;
  font-size: 14px;

  .radio {
    display: flex;
    align-items: center;
  }

  .amount {
    color: #c00000;
  }

  .btn-settle {
    height: 50px;
    min-width: 100px;
    background-color: #c00000;
    color: white;
    line-height: 50px;
    text-align: center;
    padding: 0 10px;
  }
}
</style>
```

### 4. 动态渲染已勾选商品的总数量

- cart.js-----getters--------------checkedCount:用来统计已勾选商品的总数量

  ```js
  // 勾选的商品的总数量
        checkedCount(state) {
          // 先使用 filter 方法，从购物车中过滤器已勾选的商品
          // 再使用 reduce 方法，将已勾选的商品总数量进行累加
          // reduce() 的返回值就是已勾选的商品的总数量
          return state.cart.filter(x => x.goods_state).reduce((total, item) => total += item.goods_count, 0)
        }
  ```

- 通过 `mapGetters` 辅助函数，将需要的 getters 映射到settle组件中使用

  ```js
  import { mapGetters } from 'vuex'
  computed: {
      ...mapGetters('m_cart', ['checkedCount']),
  },
  ```

- 将 `checkedCount` 的值渲染到页面中

  ```vue
  <!-- 结算按钮 -->
  <view class="btn-settle">结算({{checkedCount}})</view>
  ```

### 5. 动态渲染全选按钮的选中状态

- 使用 `mapGetters` 辅助函数，将**商品的总数量**映射到当前组件中使用，并定义一个叫做 `isFullCheck` 的计算属性

  ```js
  computed: {
      ...mapGetters('m_cart', ['checkedCount', 'total']),
  	// 2. 是否全选
      isFullCheck() {
        return this.total === this.checkedCount
      },
    },
  ```

- 为 radio 组件动态绑定 `checked` 属性的值

  ```vue
   <!-- 全选区域 -->
  <label class="radio">
     <radio color="#C00000" :checked="isFullCheck" /><text>全选</text>
   </label>
  ```

### 6. 商品的全选/反选功能

- cart.js--------------mutations---------updateAllGoodsState() :用来修改所有商品的勾选状态

  ```js
  // 6. 更新所有商品的勾选状态
        updateAllGoodsState(state, newState) {
          // 循环更新购物车中每件商品的勾选状态
          state.cart.forEach(x => x.goods_state = newState)
          // 持久化存储到本地
          this.commit('m_cart/saveToStorage')
        }
  ```

- 通过 `mapMutations` 辅助函数，将需要的 mutations 方法映射到settle组件中使用

  ```js
  import { mapGetters, mapMutations } from 'vuex'
  methods: {
  	  //  使用 mapMutations 辅助函数，把 m_cart 模块提供的 updateAllGoodsState 方法映射到当前组件中使用
        ...mapMutations('m_cart', ['updateAllGoodsState']),
    }
  ```

- 为 的 `label` 组件绑定 `click` 事件处理函数

  ```vue
  <!-- 全选区域 -->
  <label class="radio" @click="changeAllState">
    <radio color="#C00000" :checked="isFullCheck" /><text>全选</text>
  </label>
  ```

  ```js
  // label 的点击事件处理函数
        changeAllState() {
          // 修改购物车中所有商品的选中状态
          // !this.isFullCheck 表示：当前全选按钮的状态取反之后，就是最新的勾选状态
          this.updateAllGoodsState(!this.isFullCheck)
        }
  ```

### 7. 动态渲染已勾选商品的总价格

- cart.js------getters-----------checkedGoodsAmount: 用来统计已勾选商品的总价格

  ```js
  // 已勾选的商品的总价
        checkedGoodsAmount(state) {
          // 先使用 filter 方法，从购物车中过滤器已勾选的商品
          // 再使用 reduce 方法，将已勾选的商品数量 * 单价之后，进行累加
          // reduce() 的返回值就是已勾选的商品的总价
          // 最后调用 toFixed(2) 方法，保留两位小数
          return state.cart.filter(x => x.goods_state)
                           .reduce((total, item) => total += item.goods_count * item.goods_price, 0)
                           .toFixed(2)
        }
  ```

- 使用 `mapGetters` 辅助函数，把需要的 `checkedGoodsAmount` 映射到settle组件中使用

  ```js
   computed: {
      ...mapGetters('m_cart', ['total', 'checkedCount', 'checkedGoodsAmount']),
    },
  ```

- 在组件结构中，渲染已勾选的商品的总价

  ```vue
  <!-- 合计区域 -->
  <view class="amount-box">
    合计:<text class="amount">￥{{checkedGoodsAmount}}</text>
  </view>
  ```

### 8. 动态计算购物车徽标的数值

- 当修改购物车中商品的数量之后，tabBar 上的数字徽标不会自动更新

- 之前直接在数量变化后调用this.setBadge()不是很合适

- **解决方案**：改造 `mixins/tabbar-badge.js` 中的代码，使用 `watch` 侦听器，监听 `total` 总数量的变化，从而动态为 tabBar 的徽标赋值

  ```js
   watch: {
      // 监听 total 值的变化
      total() {
        // 调用 methods 中的 setBadge 方法，重新为 tabBar 的数字徽章赋值
        this.setBadge()
      },
    },
  ```

### 9. 渲染购物车为空时的页面结构

- 改造 `cart.vue` 页面的 UI 结构，使用 `v-if` 和 `v-else` 控制**购物车区域**和**空白购物车区域**的按需展示

  ```vue
  <view class="cart-container" v-if="cart.length !== 0">
     ...............
  </view>
  <!-- 空白购物车区域 -->
  <view class="empty-cart" v-else>
    <image src="/static/cart_empty@2x.png" class="empty-img"></image>
    <text class="tip-text">竟然是空的~</text>
  </view>
  ```

## 十二、登录与支付

### 零、结算条件判断

- 点击结算按钮进行条件判断

- **用户点击了结算按钮之后，需要先后判断：是否勾选了要结算的商品、是否选择了收货地址、是否登录**

  1. 在 settle组件中，为结算按钮绑定点击事件处理函数

     ```vue
     <!-- 结算按钮 -->
     <view class="btn-settle" @click="settlement">结算({{checkedCount}})</view>
     ```

     ```js
     // 点击了结算按钮
         settlement() {
           // 1. 先判断是否勾选了要结算的商品
           if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品！')
         
           // 2. 再判断用户是否选择了收货地址
           if (!this.addstr) return uni.$showMsg('请选择收货地址！')
         
           // 3. 最后判断用户是否登录了
           if (!this.token) return uni.$showMsg('请先登录！')
         }
     ```

  2. 使用 `mapGetters` 辅助函数，从 `m_user` 模块中将 `addstr` 映射到settle组件中使用
  
     ```js
     // addstr 是详细的收货地址
     ...mapGetters('m_user', ['addstr']),
     ```
  
  3. user.js--------state:声明 `token` 字符串
  
     ```js
     state: () => ({
           // 2. 读取本地的收货地址数据，初始化 address 对象
           address: JSON.parse(uni.getStorageSync('address') || '{}'),
           // 登录成功之后的 token 字符串
           token: '',
         }),
     ```
  
  4. 使用 `mapState` 辅助函数，从 `m_user` 模块中将 `token` 映射到settle组件中使用
  
     ```js
     import { mapGetters, mapMutations, mapState } from 'vuex'
     computed: {
         // token 是用户登录成功之后的 token 字符串
         ...mapState('m_user', ['token']), 
       },
     ```

### 一、登录

1. 创建组件

- 登录组件-login

- 用户信息组件-userInfo

- profile页面，通过 `mapState` 辅助函数，导入需要的 `token` 字符串

  ```js
  // 1. 从 vuex 中按需导入 mapState 辅助函数
  import { mapState } from 'vuex'
  computed: {
      // 2. 从 m_user 模块中导入需要的 token 字符串
      ...mapState('m_user', ['token']),
  },
  ```

- profile页面,实现**登录组件**和**用户信息组件**的按需展示

  ```vue
  <view>
  
      <!-- 用户未登录时，显示登录组件 -->
      <login v-if="!token"></login>
  
      <!-- 用户登录后，显示用户信息组件 -->
      <userInfo v-else></userInfo>
  
  </view>
  ```

#### 2. 登录组件布局

```vue
<template>
  <view class="login-container">
    <!-- 提示登录的图标 -->
    <uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <!-- 登录按钮 -->
    <button type="primary" class="btn-login">一键登录</button>
    <!-- 登录提示 -->
    <view class="tips-text">登录后尽享更多权益</view>
  </view>
</template>


<script>
export default {
  name: "login",
  data() {
    return {}
  },
}
</script>

<style lang="scss">
.login-container {
  // 登录盒子的样式
  height: 750rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #f8f8f8;
  position: relative;
  overflow: hidden;

  // 绘制登录盒子底部的半椭圆造型
  &::after {
    content: ' ';
    display: block;
    position: absolute;
    width: 100%;
    height: 40px;
    left: 0;
    bottom: 0;
    background-color: white;
    border-radius: 100%;
    transform: translateY(50%);
  }

  // 登录按钮的样式
  .btn-login {
    width: 90%;
    border-radius: 100px;
    margin: 15px 0;
    background-color: #c00000;
  }

  // 按钮下方提示消息的样式
  .tips-text {
    font-size: 12px;
    color: gray;
  }
}
</style>
```

#### 3. 点击登录按钮获取微信用户的基本信息

- **需要获取微信用户的头像、昵称等基本信息**

  ```vue
  <!-- 登录按钮 -->
  <button
  	 type="primary"
  	 class="btn-login"
  	 open-type="getUserInfo"
  	 @click="getUserInfo"
  >一键登录</button>
  ```

  ```js
  // 获取微信用户的基本信息
      async getUserInfo() {
        const [err, res] = await uni.getUserProfile({
              desc: '用于会员登录'
        }).catch(err => err)
        // 判断是否获取用户信息成功
  	//   console.log(err);
        if (err?.errMsg === 'getUserProfile:fail auth deny') return uni.$showMsg('您取消了登录授权！')
        // 获取用户信息成功，res.userInfo就是用户的基本信息
        console.log(res.userInfo)
      }
    }
  ```

#### 4. 将用户的基本信息存储到 vuex

- user.js------state---声明 `userinfo` 的信息对象

  ```js
  // 用户的基本信息
  userinfo: JSON.parse(uni.getStorageSync('userinfo') || '{}')
  ```

- user.js--------mutations

  ```js
   // 更新用户的基本信息
   updateUserInfo(state, userinfo) {
     state.userinfo = userinfo
     // 通过 this.commit() 方法，调用 m_user 模块下的 saveUserInfoToStorage 方法，将 userinf 象持久化存储到本地
     this.commit('m_user/saveUserInfoToStorage')
   } 
   // 将 userinfo 持久化存储到本地
   saveUserInfoToStorage(state) {
     uni.setStorageSync('userinfo', JSON.stringify(state.userinfo))
   }
  ```

- 使用 `mapMutations` 辅助函数，将需要的方法映射到 `login` 组件中使用

  ```js
  // 1. 按需导入 mapMutations 辅助函数
  import { mapMutations } from 'vuex'
   methods: {
  
  	// 2. 调用 mapMutations 辅助方法，把 m_user 模块中的 updateUserInfo 映射到当前组件中使用
      ...mapMutations('m_user', ['updateUserInfo']),
  
      // 1. 获取微信用户的基本信息
      async getUserInfo() {
        ...............
  	  // 3. 将用户的基本信息存储到 vuex 中
        this.updateUserInfo(res.userInfo)
      }
    }
  ```

#### 5. 登录获取 Token 字符串

- 当获取到了微信用户的基本信息之后，还需要进一步调用登录相关的接口，从而换取登录成功之后的 Token 字符串

- 在 `getUserInfo` 方法中，预调用 `this.getToken()` 方法，同时把获取到的用户信息传递进去

- `getToken` 方法，调用登录相关的 API，实现登录的功能

  ```js
  // 1. 获取微信用户的基本信息
      async getUserInfo() {
        ...........
  	  // 4. 获取登录成功后的 Token 字符串
        this.getToken(res)
      },
  
  	// 调用登录接口，换取永久的 token
      async getToken(info) {
        // 调用微信登录接口
        const [err, res] = await uni.login().catch(err => err)
        // 判断是否 uni.login() 调用失败
        if (err || res.errMsg !== 'login:ok') return uni.$showError('登录失败！')
        // 准备参数对象
        const query = {
           code: res.code,
           encryptedData: info.encryptedData,
           iv: info.iv,
           rawData: info.rawData,
           signature: info.signature
        }
     
         // 换取 token
         const { data: loginResult } = await uni.$http.post('/api/public/v1/users/wxlogin', query)
         if (loginResult.meta.status !== 200) return uni.$showMsg('登录失败！')
         uni.$showMsg('登录成功')
     
       }
  ```

#### 6. 将 Token 存储到 vuex

- user.js-----mutations

  ```js
  // 更新 token 字符串
  updateToken(state, token) {
    state.token = token
    // 通过 this.commit() 方法，调用 m_user 模块下的 saveTokenToStorage 方法，将 token 字久化存储到本地
    this.commit('m_user/saveTokenToStorage')
  },

  // 将 token 字符串持久化存储到本地
  saveTokenToStorage(state) {
    uni.setStorageSync('token', state.token)
  }
  ```

- user.js-----state---token

  ```js
  state: ()=> {
      // 登录成功之后的 token 字符串
      token: uni.getStorageSync('token') || '',
  }
  ```

- `updateToken` 方法映射到login组件中使用

  ```js
   methods: {
  
  	// 2. 调用 mapMutations 辅助方法，把 m_user 模块中的 updateUserInfo 映射到当前组件中使用
  	// 1. 使用 mapMutations 辅助方法，把 m_user 模块中的 updateToken 方法映射到当前组件中使用
  	...mapMutations('m_user', ['updateUserInfo', 'updateToken']),
  
      ................
  	// 调用登录接口，换取永久的 token
      async getToken(info) {
        ..............
  	   // 2. 更新 vuex 中的 token
  	   this.updateToken(loginResult.message.token)
       }
  
    }
  ```

- 因为**获取token的接口无法使用**了

  ```js
  // 调用登录接口，换取永久的 token
  async getToken(info) {
    // 调用微信登录接口
    const [err, res] = await uni.login().catch(err => err)
    // 判断是否 uni.login() 调用失败
    if (err || res.errMsg !== 'login:ok') return uni.$showError('登录失败！')
    // 准备参数对象
    const query = {
       code: res.code,
       encryptedData: info.encryptedData,
       iv: info.iv,
       rawData: info.rawData,
       signature: info.signature
    }
  
     // 换取 token
     //    const { data: loginResult } = await uni.$http.post('/api/public/v1/users/wxlogin',query)
   //    console.log(loginResult);
     //    if (loginResult.meta.status !== 200) return uni.$showMsg('登录失败！')
   
     //    uni.$showMsg('登录成功')
   // 2. 更新 vuex 中的 token
   //    this.updateToken(loginResult.message.token)
   /**
  * 因为获取token的接口无法使用了，为了做测试，
  * 可以将this.updateToken(loginResult.message.token)
  * 写成
  * this.updateToken('xunitokentokentokentoken')
  * 用一个假的虚拟token代替，并把换取token的请求和判断登录失败的两行代码注释掉
    */
   this.updateToken('xunitokentokentokentoken')
  
   }
  ```


### 二、用户信息

#### 1. 用户头像昵称区域的基本布局

- userInfo

  ```vue
  <template>
    <view class="my-userinfo-container">
  
      <!-- 头像昵称区域 -->
      <view class="top-box">
        <image src="" class="avatar"></image>
        <view class="nickname">xxx</view>
      </view>
  
    </view>
  </template>
  
  <script>
  export default {
    name: "userInfo",
    data() {
      return {}
    },
  }
  </script>
  
  <style lang="scss">
  .my-userinfo-container {
    height: 100%;
    // 为整个组件的结构添加浅灰色的背景
    background-color: #f4f4f4;
  
    .top-box {
      height: 400rpx;
      background-color: #c00000;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
  
      .avatar {
        display: block;
        width: 90px;
        height: 90px;
        border-radius: 45px;
        border: 2px solid white;
        box-shadow: 0 1px 5px black;
      }
  
      .nickname {
        color: white;
        font-weight: bold;
        font-size: 16px;
        margin-top: 10px;
      }
    }
  }
  </style>
  ```

- profile---最外层添加类名

  ```scss
  .my-container {
    height: 100%;
  }
  ```

#### 2. 渲染用户的头像和昵称数据

- 通过 `mapState` 辅助函数，将用户信息映射到userInfo组件中使用

  ```js
  / 按需导入 mapState 辅助函数
  import { mapState } from 'vuex'
  
  export default {
    name: "userInfo",
    computed: {
      // 将 m_user 模块中的 userinfo 映射到当前页面中使用
      ...mapState('m_user', ['userinfo']),
    },
    data() {
      return {}
    },
  }
  ```

- 将用户的头像和昵称渲染到页面中

  ```vue
  <!-- 头像昵称区域 -->
  <view class="top-box">
    <image :src="userinfo.avatarUrl" class="avatar"></image>
    <view class="nickname">{{userinfo.nickName}}</view>
  </view>
  ```


#### 3.  渲染第一个面板区域

- 结构

  ```vue
  <!-- 面板的列表区域 -->
  <view class="panel-list">
    <!-- 第一个面板 -->
    <view class="panel">
      <!-- panel 的主体区域 -->
      <view class="panel-body">
        <!-- panel 的 item 项 -->
        <view class="panel-item">
          <text>8</text>
          <text>收藏的店铺</text>
        </view>
        <view class="panel-item">
          <text>14</text>
          <text>收藏的商品</text>
        </view>
        <view class="panel-item">
          <text>18</text>
          <text>关注的商品</text>
        </view>
        <view class="panel-item">
          <text>84</text>
          <text>足迹</text>
        </view>
      </view>
    </view>
  
    <!-- 第二个面板 -->
    
  
    <!-- 第三个面板 -->
  </view>
  ```

- 样式

  ```scss
  /*面板 */
  .panel-list {
    padding: 0 10px;
    position: relative;
    top: -10px;
  
    .panel {
      background-color: white;
      border-radius: 3px;
      margin-bottom: 8px;
  
      .panel-body {
        display: flex;
        justify-content: space-around;
  
        .panel-item {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: space-around;
          font-size: 13px;
          padding: 10px 0;
        }
      }
    }
  }
  ```


#### 4. 渲染第二个面板区域

- 结构

  ```vue
  <!-- 第二个面板 -->
  <view class="panel">
      <!-- 面板的标题 -->
      <view class="panel-title">我的订单</view>
      <!-- 面板的主体 -->
      <view class="panel-body">
        <!-- 面板主体中的 item 项 -->
        <view class="panel-item">
          <image src="/static/my-icons/icon1.png" class="icon"></image>
          <text>待付款</text>
        </view>
        <view class="panel-item">
          <image src="/static/my-icons/icon2.png" class="icon"></image>
          <text>待收货</text>
        </view>
        <view class="panel-item">
          <image src="/static/my-icons/icon3.png" class="icon"></image>
          <text>退款/退货</text>
        </view>
        <view class="panel-item">
          <image src="/static/my-icons/icon4.png" class="icon"></image>
          <text>全部订单</text>
        </view>
      </view>
    </view>
  ```

- 样式

  ```scss
  /*面板 */
  .panel-list {
    padding: 0 10px;
    position: relative;
    top: -10px;
  
    .panel {
      background-color: white;
      border-radius: 3px;
      margin-bottom: 8px;
  
      .panel-title {
        line-height: 45px;
        padding-left: 10px;
        font-size: 15px;
        border-bottom: 1px solid #f4f4f4;
      }
  
      .panel-body {
        display: flex;
        justify-content: space-around;
  
        .panel-item {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: space-around;
          font-size: 13px;
          padding: 10px 0;
  
          .icon {
            width: 35px;
            height: 35px;
          }
        }
      }
    }
  }
  ```


#### 5. 渲染第三个面板区域

- 结构

  ```vue
  <!-- 第三个面板 -->
  <view class="panel">
    <view class="panel-list-item">
      <text>收货地址</text>
      <uni-icons type="arrowright" size="15"></uni-icons>
    </view>
    <view class="panel-list-item">
      <text>联系客服</text>
      <uni-icons type="arrowright" size="15"></uni-icons>
    </view>
    <view class="panel-list-item">
      <text>退出登录</text>
      <uni-icons type="arrowright" size="15"></uni-icons>
    </view>
  </view>
  ```

- 样式

  ```scss
  .panel-list-item {
    height: 45px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 15px;
    padding: 0 10px;
  }
  ```

#### 6. 退出登录的功能

- 第三个面板区域中的 `退出登录` 项绑定 `click` 点击事件处理函数

  ```vue
  <view class="panel-list-item" @click="logout">
    <text>退出登录</text>
    <uni-icons type="arrowright" size="15"></uni-icons>
  </view>
  ```

  ```js
  // 按需导入辅助函数
  import { mapState, mapMutations } from 'vuex'
  
  export default {
    ..................
    methods: {
  	  ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateAddress']),
  	  
  	  // 退出登录
        async logout() {
          // 询问用户是否退出登录
          const [err, succ] = await uni.showModal({
            title: '提示',
            content: '确认退出登录吗？'
          }).catch(err => err)
        
          if (succ && succ.confirm) {
             // 用户确认了退出登录的操作
             // 需要清空 vuex 中的 userinfo、token 和 address
             this.updateUserInfo({})
             this.updateToken('')
             this.updateAddress({})
          }
        },
    }
  }
  ```

### 三、3秒后自动跳转

- **在购物车页面，当用户点击 “结算” 按钮时，****如果用户没有登录，则 3 秒后自动跳转到登录页面**

##### 1. 延时定时器跳转

1. settle组件-----methods---声明一个叫做 `showTips` 的方法，专门用来展示倒计时的提示消息

   ```js
   // 展示倒计时的提示消息
   showTips(n) {
     // 调用 uni.showToast() 方法，展示提示消息
     uni.showToast({
       // 不展示任何图标
       icon: 'none',
       // 提示的消息
       title: '请登录后再结算！' + n + ' 秒后自动跳转到登录页',
       // 为页面添加透明遮罩，防止点击穿透
       mask: true,
       // 1.5 秒后自动消失
       duration: 1500
     })
   }
   ```

2. 在 `data` 节点中声明倒计时的秒数

   ```js
    data() {
       return {
         // 倒计时的秒数
         seconds: 3
       }
     },
   ```

3. settlement（）--如果用户没有登录，则**预调用**一个叫做 `delayNavigate` 的方法，进行倒计时的导航跳转

   ```js
   // 点击了结算按钮
   settlement() {
     // 1. 先判断是否勾选了要结算的商品
     if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品！')
   
     // 2. 再判断用户是否选择了收货地址
     if (!this.addstr) return uni.$showMsg('请选择收货地址！')
   
     // 3. 最后判断用户是否登录了
     // if (!this.token) return uni.$showMsg('请先登录！') 
     // 3. 最后判断用户是否登录了，如果没有登录，则调用 delayNavigate() 进行倒计时的导航跳转
     if (!this.token) return this.delayNavigate()
   },
   ```

4. `delayNavigate` 方法，初步实现**倒计时的提示功能**

   ```js
   // 延迟导航到 profile 页面
   delayNavigate() {
     // 1. 展示提示消息，此时 seconds 的值等于 3
     this.showTips(this.seconds)
   
     // 2. 创建定时器，每隔 1 秒执行一次
     setInterval(() => {
       // 2.1 先让秒数自减 1
       this.seconds--
       // 2.2 再根据最新的秒数，进行消息提示
       this.showTips(this.seconds)
     }, 1000)
   },
   ```


##### 2. 问题1

- 延时器： 定时器不会自动停止，此时秒数会出现等于 0 或小于 0 的情况！

- 在 `data` 节点中声明定时器的 Id

  ```js
  data() {
    return {
      // 倒计时的秒数
      seconds: 3,
      // 定时器的 Id
      timer: null
    }
  },
  ```

- 改造 `delayNavigate` 方法

  ```js
  // 延迟导航到 profile 页面
  delayNavigate() {
    // 1. 展示提示消息，此时 seconds 的值等于 3
    this.showTips(this.seconds)
  
    // 1. 将定时器的 Id 存储到 timer 中
    this.timer = setInterval(() => {
      this.seconds--
  
      // 2. 判断秒数是否 <= 0
      if (this.seconds <= 0) {
        // 2.1 清除定时器
        clearInterval(this.timer)
  
        // 2.2 跳转到 my 页面
        uni.switchTab({
          url: '/pages/profile/profile'
        })
  
        // 2.3 终止后续代码的运行（当秒数为 0 时，不再展示 toast 提示消息）
        return
      }
  
      this.showTips(this.seconds)
    }, 1000)
  },
  ```


##### 3. 问题2

- 倒计时3秒数不会被重置，导致第 2 次，3 次，n 次 的倒计时跳转功能无法正常工作，直接就跳

- `delayNavigate` 方法，在执行此方法时，先将 `seconds` 秒数重置为 `3` 即可

  ```js
  // 延迟导航到 profile 页面
  delayNavigate() {
    // 把 data 中的秒数重置成 3 秒
    this.seconds = 3
    .
  },
  ```


##### 4. 登录成功之后再返回之前的页面

> 核心实现思路：在自动跳转到登录页面成功之后，把返回页面的信息存储到 vuex 中，从而方便登录成功之后，根据返回页面的信息重新跳转回去
>
> 返回页面的信息对象，主要包含 { openType, from } 两个属性，其中 openType 表示以哪种方式导航回之前的页面；from 表示之前页面的 url 地址

1. user.js-----state: 声明重定向 `redirectInfo` 对象

   ```js
   // 重定向的 object 对象 { openType, from }
   redirectInfo: null
   ```

2. user.js-----mutations-----updateRedirectInfo:更新重定向信息

   ```js
   // 更新重定向的信息对象
   updateRedirectInfo(state, info) {
      state.redirectInfo = info
   }
   ```

3. 通过 `mapMutations` 辅助方法，把 `m_user` 模块中的 `updateRedirectInfo` 方法映射到settle页面中使用

   ```js
   methods: {
     // 把 m_user 模块中的 updateRedirectInfo 方法映射到当前页面中使用
     ...mapMutations('m_user', ['updateRedirectInfo']),
   }
   ```

4. `delayNavigate` 方法，当成功跳转到 `profile 页面` 之后，将重定向的信息对象存储到 vuex 中

   ```js
   // 延迟导航到 profile 页面
       delayNavigate() {
         ..................
             // 2.2 跳转到 profile 页面
             uni.switchTab({
               url: '/pages/profile/profile',
               // 页面跳转成功之后的回调函数
               success: () => {
                 // 调用 vuex 的 updateRedirectInfo 方法，把跳转信息存储到 Store 中
                 this.updateRedirectInfo({
                   // 跳转的方式
                   openType: 'switchTab',
                   // 从哪个页面跳转过去的
                   from: '/pages/cart/cart'
                 })
               }
             })
       
             .................
           }
       
           .............
         }, 1000)
       },
   ```

5. 通过 `mapState` 和 `mapMutations` 辅助方法，将 vuex 中需要的数据和方法，映射到login页面中使用

   ```js
   // 按需导入辅助函数
   import { mapMutations, mapState } from 'vuex'
   
   export default {
     computed: {
       // 调用 mapState 辅助方法，把 m_user 模块中的数据映射到当前用组件中使用
       ...mapState('m_user', ['redirectInfo']),
     },
     methods: {
       // 调用 mapMutations 辅助方法，把 m_user 模块中的方法映射到当前组件中使用
       ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
     },
   }
   ```

6. `getToken` 方法，当登录成功之后，预调用 `this.navigateBack()` 方法返回登录之前的页面

   ```js
   // 调用登录接口，换取永久的 token
   async getToken(info) {
     // 省略其它代码...
   
     // 判断 vuex 中的 redirectInfo 是否为 null
     // 如果不为 null，则登录成功之后，需要重新导航到对应的页面
     this.navigateBack()
   }
   
   // 返回登录之前的页面
   navigateBack() {
     // redirectInfo 不为 null，并且导航方式为 switchTab
     if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
       // 调用小程序提供的 uni.switchTab() API 进行页面的导航
       uni.switchTab({
         // 要导航到的页面地址
         url: this.redirectInfo.from,
         // 导航成功之后，把 vuex 中的 redirectInfo 对象重置为 null
         complete: () => {
           this.updateRedirectInfo(null)
         }
       })
     }
   }
   ```

### 四、支付

#### 1. 在请求头中添加 Token 身份认证的字段

> **只有在登录之后才允许调用支付相关的接口**，所以必须为有权限的接口添加身份认证的请求头字段

1. main.js, 改造 $http.beforeRequest 请求拦截器中的代码

   ```js
   // 请求拦截器
   $http.beforeRequest = function (options) {
     uni.showLoading({
       title: '数据加载中...',
     })
   
     // 判断请求的是否为有权限的 API 接口
     if (options.url.indexOf('/profile/') !== -1) {
       // 为请求头添加身份认证字段
       options.header = {
         // 字段的值可以直接从 vuex 中进行获取
         Authorization: store.state.m_user.token,
       }
     }
   }
   ```

#### 2. 微信支付的流程

1. **创建订单**
   - 请求创建订单的 API 接口：把（订单金额、收货地址、订单中包含的商品信息）发送到服务器
   - 服务器响应的结果：*订单编号*
2. **订单预支付**
   - 请求订单预支付的 API 接口：把（订单编号）发送到服务器
   - 服务器响应的结果：*订单预支付的参数对象*，里面包含了订单支付相关的必要参数
3. **发起微信支付**
   - 调用 `uni.requestPayment()` 这个 API，发起微信支付；把步骤 2 得到的 “订单预支付对象” 作为参数传递给 `uni.requestPayment()` 方法
   - 监听 `uni.requestPayment()` 这个 API 的 `success`，`fail`，`complete` 回调函数

#### 3. 创建订单

- settle组件：`settlement` 方法，当前三个判断条件通过之后，调用实现微信支付的方法

  ```js
  // 点击了结算按钮
  settlement() {
   ............
    // 4. 实现微信支付功能
    this.payOrder()
  },
  // 微信支付
  async payOrder() {
    // 1. 创建订单
    // 1.1 组织订单的信息对象
    const orderInfo = {
      // 开发期间，注释掉真实的订单价格，
      // order_price: this.checkedGoodsAmount,
      // 写死订单总价为 1 分钱
      order_price: 0.01,
      consignee_addr: this.addstr,
      goods: this.cart.filter(x => x.goods_state).map(x => ({ goods_id: x.goods_idgoods_number: x.goods_count, goods_price: x.goods_price }))
    }
    // 1.2 发起请求创建订单
    const { data: res } = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
    if (res.meta.status !== 200) return uni.$showMsg('创建订单失败！')
    // 1.3 得到服务器响应的“订单编号”
    const orderNumber = res.message.order_number
  
    // 2. 订单预支付
  
     // 3. 发起微信支付
   }
  ```


#### 4. 订单预支付

- `settlement` 方法

  ```js
  // 微信支付
     async payOrder() {
       // 1. 创建订单
       // 1.1 组织订单的信息对象
       const orderInfo = {
         // 开发期间，注释掉真实的订单价格，
         // order_price: this.checkedGoodsAmount,
         // 写死订单总价为 1 分钱
         order_price: 0.01,
         consignee_addr: this.addstr,
         goods: this.cart.filter(x => x.goods_state).map(x => ({ goods_id: x.goods_id, goods_number: x.goods_count, goods_price: x.goods_price }))
       }
       // 1.2 发起请求创建订单
       const { data: res } = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
       if (res.meta.status !== 200) return uni.$showMsg('创建订单失败！')
       // 1.3 得到服务器响应的“订单编号”
       const orderNumber = res.message.order_number
     
        // 2. 订单预支付
        // 2.1 发起请求获取订单的支付信息
        const { data: res2 } = await uni.$http.post('/api/public/v1/my/orders/req_unifiedorder', {   order_number: orderNumber })
        // 2.2 预付订单生成失败
        if (res2.meta.status !== 200) return uni.$showError('预付订单生成失败！')
        // 2.3 得到订单支付相关的必要参数
        const payInfo = res2.message.pay
    
     
        // 3. 发起微信支付
      }
  ```
  

#### 5. 发起微信支付

      ```js
      // 微信支付
   async payOrder() {
     // 1. 创建订单
     // 1.1 组织订单的信息对象
     const orderInfo = {
       // 开发期间，注释掉真实的订单价格，
       // order_price: this.checkedGoodsAmount,
       // 写死订单总价为 1 分钱
       order_price: 0.01,
       consignee_addr: this.addstr,
       goods: this.cart.filter(x => x.goods_state).map(x => ({ goods_id: x.goods_id, goods_number: x.goods_count, goods_price: x.goods_price }))
     }
     // 1.2 发起请求创建订单
     const { data: res } = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
     if (res.meta.status !== 200) return uni.$showMsg('创建订单失败！')
     // 1.3 得到服务器响应的“订单编号”
     const orderNumber = res.message.order_number

      // 2. 订单预支付
      // 2.1 发起请求获取订单的支付信息
      const { data: res2 } = await uni.$http.post('/api/public/v1/my/orders/req_unifiedorder', {   order_number: orderNumber })
      // 2.2 预付订单生成失败
      if (res2.meta.status !== 200) return uni.$showError('预付订单生成失败！')
      // 2.3 得到订单支付相关的必要参数
      const payInfo = res2.message.pay

   

      // 3. 发起微信支付
      // 3.1 调用 uni.requestPayment() 发起微信支付
      const [err, succ] = await uni.requestPayment(payInfo)
      // 3.2 未完成支付
      if (err) return uni.$showMsg('订单未支付！')
      // 3.3 完成了支付，进一步查询支付的结果
      const { data: res3 } = await uni.$http.post('/api/public/v1/my/orders/chkOrder', { order_number: orderNumber })
      // 3.4 检测到订单未支付
      if (res3.meta.status !== 200) return uni.$showMsg('订单未支付！')
      // 3.5 检测到订单支付完成
      uni.showToast({
        title: '支付完成！',
        icon: 'success'
      })
      
    }
      ```



## 发布

### 1. 为什么发布

1. 小程序只有发布之后，才能被用户搜索并使用
2. 开发期间的小程序为了便于调试，含有 sourcemap 相关的文件，并且代码没有被压缩，因此体积较大，不适合直接当作线上版本进行发布
3. 通过执行 “小程序发布”，能够优化小程序的体积，提高小程序的运行性能

### 2. 发布流程

1. 点击 `HBuilderX` 菜单栏上的 `发行` -> `小程序-微信(仅适用于uni-app)`
2. 在弹出框中填写要发布的**小程序的名称**和**AppId**之后，点击发行按钮
3. 在 `HBuilderX` 的控制台中**查看小程序发布编译的进度**
4. 发布编译完成之后，会自动打开一个新的**微信开发者工具界面**，此时，点击工具栏上的上传按钮
5. 填写**版本号**和**项目备注**之后，点击**上传**按钮
6. 上传完成之后，会出现提示消息，直接点击**确定**按钮即可
7. 通过微信开发者工具上传的代码，默认处于**版本管理**的**开发版本**列表中
8. 将 `开发版本提交审核` -> 再将 `审核通过的版本发布上线`，即可实现小程序的发布和上线
