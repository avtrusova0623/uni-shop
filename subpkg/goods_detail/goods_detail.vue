<template>
  <view v-if="goodsInfo.goods_name">
    <!-- 轮播图 -->
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

    <!-- 商品详情介绍信息 -->
    <rich-text :nodes="goodsInfo.goods_introduce"></rich-text>

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


  </view>
</template>

<script>
// 从 vuex 中按需导出 mapState 辅助方法
import { mapState , mapMutations, mapGetters} from 'vuex'

export default {
  data() {
    return {
      // 商品信息
      goodsInfo: {},
      // 左侧按钮组的配置对象
      options: [
        {
          icon: 'chat',
          text: '客服'
        }, {
          icon: 'cart',
          text: '购物车',
          // info: 2
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
    }
  },

  onLoad(options) {
    // 1. 获取商品id
    const  id = options.goods_id

    // 2. 请求商品详情数据
    this.getGoodsDetail(id)
  },

  computed: {
    // 调用 mapState 方法，把 m_cart 模块中的 cart 数组映射到当前页面中，作为计算属性来使用
    // ...mapState('模块的名称', ['要映射的数据名称1', '要映射的数据名称2'])
    ...mapState('m_cart', ['cart']),
    ...mapGetters('m_cart', ['total']),
  },

  methods: {
    // 把 m_cart 模块中的 addToCart 方法映射到当前页面使用
    ...mapMutations('m_cart', ['addToCart']),


    // 1. 请求商品数据
    async getGoodsDetail(id) {
      const { data: res } = await uni.$http.get("/api/public/v1/goods/detail", {
       goods_id: id
      })
      if (res.meta.status !== 200) return uni.$showMsg()

     
      // 使用字符串的 replace() 方法，将 webp 的后缀名替换为 jpg 的后缀名
      res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g, '<img style="display:block;" ').replace(/webp/g, 'jpg')

      // 为 data 中的数据赋值
      this.goodsInfo = res.message
    },

    // 2. 图片预览
    preview(index) {
       // 调用 uni.previewImage() 方法预览图片
       uni.previewImage({
         // 预览时，默认显示图片的索引
         current: index,
         // 所有图片 url 地址的数组
         urls: this.goodsInfo.pics.map(x => x.pics_big)
       })
    },

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
         this.addToCart(goods)

         uni.$showMsg("商品添加成功")
   
      }
    },

  },

  watch: {
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

  },


};
</script>

<style lang="scss">
/* 轮播图 */
swiper {
  height: 750rpx;

  image {
    width: 100%;
    height: 100%;
  }
}

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
</style>
