<template>
  <view>
    <!-- 搜索 -->
    <searchBar @click.native="goSearchPage"></searchBar>
    <!-- 分类滑动区 -->
    <view class="scroll-view-container">
      <!-- 左侧的滚动视图区域 -->
      <scroll-view class="left-scroll-view" scroll-y :style="{ height: wh + 'px' }">
        <block v-for="(leftDatas,indexl) in leftCateList" :key="indexl">
          <view :class="['left-scroll-view-item', indexl === active ? 'active':'']"
            @click="handleLeftClick(indexl)"
          >{{leftDatas.cat_name}}</view>
        </block>
      </scroll-view>
      <!-- 右侧的滚动视图区域 -->
  	  <scroll-view class="right-scroll-view" 
      scroll-y 
      :style="{height: wh + 'px'}"
      :scroll-top="scrollTop"
      >
        <view class="cate-lv2" v-for="(rightDatas, index2) in rightCateList" :key="index2">
          <!-- 标题 -->
          <view class="cate-lv2-title">/ {{rightDatas.cat_name}} /</view>
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
        </view>
      </scroll-view>
    </view>
  </view>
  
</template>

<script>
import searchBar from '../../components/searchBar/searchBar.vue';
// 导入自己封装的 mixin 模块
import badgeMix from '@/mixins/tabbar-badge.js'

export default {
  components: { searchBar },
  data() {
    return {
      // 左侧分类数据
      leftCateList: [],
      // 右侧二级分类数据
      rightCateList: [],
	    // 窗口的可用高度 = 屏幕高度 - navigationBar高度 - tabBar 高度
      wh: 0,
      // 当前选中项的索引，默认让第一项被选中
      active: 0,
      // 滚动条距离顶部的距离
      scrollTop: 0
      
    };
  },
  mixins: [badgeMix],
  onLoad() {
    // 1. 获取分类数据
    this.getCateList();

    // 获取当前系统的信息
    const sysInfo = uni.getSystemInfoSync();
    // 可用高度 = 屏幕高度 - navigationBar高度 - tabBar高度 - 自定义的search组件高度
    this.wh = sysInfo.windowHeight - 50
  },
  methods: {
    // 1. 请求分类数据
    async getCateList() {
      const { data: res } = await uni.$http.get("/api/public/v1/categories");
      if (res.meta.status !== 200) return uni.$showMsg();
      console.log(res);
      this.leftCateList = res.message;
      // 获取右侧分类数据，默认显示左侧第一项children数据
      this.rightCateList = res.message[0].children
    },

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

    // 3. 右侧分类点击事件
    goGoodsList(id) {
      // 跳转页面
      uni.navigateTo({ 
        url: `/subpkg/goods_list/goods_list?cat_id=${id}`
      })

    },

    // 4. 跳转到搜索页面
    goSearchPage() {
      uni.navigateTo({
        url: `/subpkg/search/search`
      })
    }
  },
};
</script>

<style lang="scss">
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
</style>
