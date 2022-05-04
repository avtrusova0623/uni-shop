<template>
  <view class="my-container">

    <!-- 用户未登录时，显示登录组件 -->
    <login v-if="!token"></login>

    <!-- 用户登录后，显示用户信息组件 -->
    <userInfo v-else></userInfo>

  </view>
</template>

<script>
// 导入自己封装的 mixin 模块
import badgeMix from '@/mixins/tabbar-badge.js'
// 1. 从 vuex 中按需导入 mapState 辅助函数
import { mapState } from 'vuex'
export default {
  data() {
    return {};
  },
  // 将 badgeMix 混入到当前的页面中进行使用
  mixins: [badgeMix],
  computed: {
    // 2. 从 m_user 模块中导入需要的 token 字符串
    ...mapState('m_user', ['token']),
  },
};
</script>

<style lang="scss">
.my-container {
  height: 100%;
}
</style>
