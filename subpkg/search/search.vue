<template>
  <view>
    <!-- 搜索框 -->
    <view class="search-box">
      <uni-search-bar
        cancelButton="none"
        :radius="100"
        placeholder="请输入搜索内容"
        @input="valueChange"
      ></uni-search-bar>
    </view>

    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchSuggestions.length !== 0">
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

    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clearHistory"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" 
        v-for="(item, index) in histories" 
        :key="index"
        @click="goGoodsList(item)"
        ></uni-tag>
      </view>
    </view>

  </view>
</template>

<script>
export default {
  data() {
    return {
      // 定时器
      timer: null,
      // 搜索关键字
      keywords: "",
      // 搜索建议结果
      searchSuggestions: [],
      // 搜索历史
      historyList: []
    };
  },
  onLoad() {
    // 加载本地存储的搜索历史记录
    this.historyList = JSON.parse(uni.getStorageSync('keywords') || '[]')
  },
  methods: {
    // 1. 输入改变事件
    valueChange(inputValue) {
      // 清除 timer 对应的延时器
      clearTimeout(this.timer);
      // 重新启动一个延时器，并把 timerId 赋值给 this.timer
      this.timer = setTimeout(() => {
        // 如果 500 毫秒内，没有触发新的输入事件，则为搜索关键词赋值
        this.keywords = inputValue;
        // console.log(inputValue)

        // 2. 根据关键词，查询搜索建议列表
        this.getSearchSuggestions();
      }, 500);
    },

    // 2. 根据搜索关键词，搜索商品建议列表
    async getSearchSuggestions() {
      // 判断关键词是否为空
      if (this.keywords === "") {
        this.searchSuggestions = [];
        return;
      }

      // 请求数据
      const res = await uni.$http.get("/api/public/v1/goods/qsearch", {
        query: this.keywords,
      });
      if (res.data.meta.status !== 200) return uni.$showMsg();
      this.searchSuggestions = res.data.message;

      // 4. 保存搜索历史数据
      this.saveSearchHistory()
    },

	// 3. 跳转到商品详情页面
	goDetail(goods_id) {
	  uni.navigateTo({
	    // 指定详情页面的 URL 地址，并传递 goods_id 参数
	    url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
	  })
	},

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
  },

  // 清空历史记录
  clearHistory() {
    // 清空 data 中保存的搜索历史
    this.historyList = []
    // 清空本地存储中记录的搜索历史
    uni.setStorageSync('keywords', '[]')
  },

  // 页面跳转，商品列表页
  goGoodsList(keywords) {
    uni.navigateTo({
      url: `/subpkg/goods_list/goods_list?query=${keywords}`
    })
  }
  },

  computed: {
    // 历史记录反序
    // reverse() 方法将数组中元素的位置颠倒,该方法会改变原数组
    // 以不要直接基于原数组调用 reverse 方法，以免修改原数组中元素的顺序
    histories() {
      let arr = [...this.historyList]
      return arr.reverse()
    }
  }

};
</script>

<style lang="scss">
/*搜索框 */
.search-box {
  position: sticky;
  top: 0;
  z-index: 999;
}
/*搜索建议 */
.sugg-list {
  padding: 0 5px;

  .sugg-item {
    font-size: 12px;
    padding: 13px 0;
    border-bottom: 1px solid #efefef;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .goods-name {
      // 文字不允许换行（单行文本）
      white-space: nowrap;
      // 溢出部分隐藏
      overflow: hidden;
      // 文本溢出后，使用 ... 代替
      text-overflow: ellipsis;
      margin-right: 3px;
    }
  }
}
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
</style>
