<template>
  <view class="login-container">
    <!-- 提示登录的图标 -->
    <uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <!-- 登录按钮 -->
    <button
	 type="primary"
	 class="btn-login"
	 open-type="getUserInfo"
	 @click="getUserInfo"
	>一键登录</button>
    <!-- 登录提示 -->
    <view class="tips-text">登录后尽享更多权益</view>
  </view>
</template>


<script>
// 1. 按需导入 mapMutations 辅助函数
import { mapMutations, mapState } from 'vuex'

export default {
  name: "login",
  data() {
    return {}
  },

  computed: {
    // 调用 mapState 辅助方法，把 m_user 模块中的数据映射到当前用组件中使用
    ...mapState('m_user', ['redirectInfo']),
  },

  methods: {

	// 2. 调用 mapMutations 辅助方法，把 m_user 模块中的 updateUserInfo 映射到当前组件中使用
	// 1. 使用 mapMutations 辅助方法，把 m_user 模块中的 updateToken 方法映射到当前组件中使用
	...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),

    // 1. 获取微信用户的基本信息
    async getUserInfo() {
      const [err, res] = await uni.getUserProfile({
            desc: '用于会员登录'
      }).catch(err => err)

      //  判断是否获取用户信息成功
	//   console.log(err);
      if (err?.errMsg === 'getUserProfile:fail auth deny') return uni.$showMsg('您取消了登录授权！')
      // 获取用户信息成功，res.userInfo就是用户的基本信息
    //   console.log(res.userInfo)

	  // 3. 将用户的基本信息存储到 vuex 中
      this.updateUserInfo(res.userInfo)

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
       //    const { data: loginResult } = await uni.$http.post('/api/public/v1/users/wxlogin', query)
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

     // 判断 vuex 中的 redirectInfo 是否为 null
     // 如果不为 null，则登录成功之后，需要重新导航到对应的页面
     this.navigateBack()
   
     },

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

  }
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