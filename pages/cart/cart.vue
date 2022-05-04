<template>
  <view class="cart-container" v-if="cart.length !== 0">
    <!-- 收货地址 -->
    <Address></Address>

    <!-- 购物车商品列表的标题区域 -->
    <view class="cart-title">
      <!-- 左侧的图标 -->
      <uni-icons type="shop" size="18"></uni-icons>
      <!-- 描述文本 -->
      <text class="cart-title-text">购物车</text>
    </view>
    <!-- 商品列表区域 -->
    <uni-swipe-action>
       <block v-for="(goods, index) in cart" :key="index">
         <!-- uni-swipe-action-item 可以为其子节点提供滑动操作的效果。需要通过 options 属性来指定操作按钮的配置信息 -->
         <uni-swipe-action-item :right-options="options" @click="swipeActionClickHandler(goods)">
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

    <!-- 结算区域 -->
    <Settle></Settle>
  </view>

  <!-- 空白购物车区域 -->
  <view class="empty-cart" v-else>
    <image src="/static/cart_empty@2x.png" class="empty-img"></image>
    <text class="tip-text">竟然是空的~</text>
  </view>
</template>

<script>
// 导入自己封装的 mixin 模块
import badgeMix from '@/mixins/tabbar-badge.js'
// 按需导入 mapState 这个辅助函数
import { mapState, mapMutations } from 'vuex'
import goodsListItem from '../../components/goods-list-item/goods-list-item.vue'
import Address from '../../components/address/address.vue'
import Settle from '../../components/settle/settle.vue'

export default {
  components: { goodsListItem, Address ,Settle},
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
  computed: {
     // 将 m_cart 模块中的 cart 数组映射到当前页面中使用
     ...mapState('m_cart', ['cart']),
   },

  // 将 badgeMix 混入到当前的页面中进行使用
  mixins: [badgeMix],
  onLoad(options) {
  },
  methods: {
    ...mapMutations('m_cart', ['updateGoodsState','updateGoodsCount','removeGoodsById']),


    // 1. 切换勾选状态
    radioChangeHandler(e) {
      // 输出得到的数据 -> {goods_id: xx, goods_state: false}
      // console.log(e);
      this.updateGoodsState(e)
    },

    // 2. 商品的数量发生了变化
    numberChangeHandler(e) {
      // console.log(e)
      this.updateGoodsCount(e)
      // tabbar购物车数量更新---mixins--- setBadge()
      // this.setBadge()
      // mixins中监听total变化
    },

    // 3. 点击了滑动操作按钮
    swipeActionClickHandler(goods) {
      // console.log(goods)
      this.removeGoodsById(goods.goods_id)
      // tabbar购物车数量更新---mixins--- setBadge()
      // this.setBadge()
    },
  
  }
}
</script>

<style lang="scss">
.cart-container {
  padding-bottom: 50px;
}

/*标题 */
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

/*购物车为空 */
.empty-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 150px;

  .empty-img {
    width: 90px;
    height: 90px;
  }

  .tip-text {
    font-size: 12px;
    color: gray;
    margin-top: 15px;
  }
}
</style>
