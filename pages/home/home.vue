<template>
  <view>
    <!-- 搜索框 -->
    <view class="search-box">
      <searchBar @click.native="goSearchPage"></searchBar>
    </view>
    
    <!-- 
      因为点击事件无效暂时使用uni-swiper
      轮播图 
		list： 轮播图数据
		indicator： 是否显示面板指示器
		indicatorMode： 指示器模式
		circular： 播放到末尾后是否重新回到开头
		nterval:	滑块自动切换时间间隔（ms）
	-->
    <!-- <u-swiper 
	 :list="swiperList" 
	 keyName="image_src" 
	 :interval="3000"
	 :loading='loading'
	 indicator 
	 indicatorMode="dot" 
	 circular
   @click="click(index)">
	 </u-swiper> -->

    <!-- 轮播图 -->
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
          :url="'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id"
        >
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>

    <!-- 分类导航 -->
    <view class="nav-list">
      <view
        class="nav-item"
        v-for="(item, index) in navList"
        :key="index"
        @click="navClickHandler(item)"
      >
        <image :src="item.image_src" class="nav-img"></image>
      </view>
    </view>

    <!-- 展示区 -->
    <view class="floor-list">
      <view class="floor-item" v-for="(item, i) in showGoodsList" :key="i">
        <!-- 标题 -->
        <image :src="item.floor_title.image_src" class="floor-title"></image>
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
      </view>
    </view>
  </view>
</template>

<script>
// 导入自己封装的 mixin 模块
import badgeMix from '@/mixins/tabbar-badge.js'
export default {
  data() {
    return {
      // 轮播图数据，初始为空
      swiperList: [],
      // 分类导航数据
      navList: [],
      // 商品展示区
      showGoodsList: [],
    };
  },
  mixins: [badgeMix],
  onLoad(options) {
    // 1. 获取轮播图数据
    this.getSwiperData();
    // 2. 获取分类导航数据
    this.getNavList();
    // 3. 获取商品展示数据
    this.getShowGoodsList();
  },
  methods: {
    // 1. 请求轮播图方法
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
        return uni.$showMsg();
      }
      // 1.3 请求成功，为 data 中的数据赋值
      this.swiperList = res.message;
      // uni.$showMsg("请求轮播图数据成功")
    },

    // 2. 请求分类导航数据
    async getNavList() {
      const { data: res } = await uni.$http.get("/api/public/v1/home/catitems");
      if (res.meta.status !== 200) return uni.$showMsg();
      this.navList = res.message;
    },

    // 3. 处理分类导航点击事件
    navClickHandler(item) {
      // 判断点击的是哪个 nav
      if (item.name === "分类") {
        uni.switchTab({
          url: "/pages/cate/cate",
        });
      }
    },

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
            "/subpkg/goods_list/goods_list?" + prod.navigator_url.split('?')[1];
        });
      });
      this.showGoodsList = res.message;
    },

    // 5. 跳转到搜索页面
    goSearchPage() {
      uni.navigateTo({
        url: `/subpkg/search/search`
      })
    }
  },
};
</script>

<style lang="scss">
/* 搜索 */
.search-box {
  // 设置定位效果为“吸顶”
  position: sticky;
  // 吸顶的“位置”
  top: 0;
  // 提高层级，防止被轮播图覆盖
  z-index: 999;
}

/* 轮播图 */
swiper {
  height: 330rpx;

  .swiper-item,
  image {
    width: 100%;
    height: 100%;
  }
}
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
</style>
