<template>
  <div class="login-container">
    <!-- 导航栏 -->
    <van-nav-bar title="登录" class="page-nav-bar" />
    <!-- 登陆表单 -->
    <van-form 
    ref="loginForm"
     @submit="onSubmit">
        <!-- 手机 -->
  <van-field
    v-model="user.mobile"
    name="mobile"
     type="number"
    maxlength="11"
    placeholder="请输入手机号码"
    :rules="userFormRules.mobile"
  >
  <i slot="left-icon" class="toutiao toutiao-shouji"></i>
</van-field>
  <!-- 验证码 -->
  <van-field
    v-model="user.code"
    name="code"
    type="number"
    maxlength="6"
    placeholder="请输入验证码"
     :rules="userFormRules.code"
  >
   <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
   <!-- 发送验证码按钮 -->
    <template #button>
        <!-- 倒计时 -->
        <van-count-down 
        v-if="isCountDownShow"
        :time="1000*5" format="ss" 
        @finish="isCountDownShow=false"
        />
        <!-- 发送验证码按钮 -->
    <van-button
    v-else
    native-type="button"
    class="send-sms-btn"
     size="small" 
     type="primary"
     @click="onSendSms">发送验证码</van-button>
  </template>
  </van-field>
  <div class="login-btn-wrap" style="margin: 16px;">
    <van-button 
    class="login-btn"
     block type="info" native-type="submit">登录</van-button>
  </div>
</van-form>
  </div>
</template>

<script>
import {login,sendSms} from '@/api/user.js'

export default {
    name:'Login',
    data() {
        return {
            user: {
                mobile: '13774074011',
                code:'246810'
            },
            userFormRules: {
                mobile: [
                    {
                        required: true,
                        message:'手机号不能为空'
                    },
                    {
                        pattern: /^1[3|5|7|8|9]\d{9}$/,
                        message:'手机号格式错误'
                    }
                ],
                code: [
                    {
                        required: true,
                        message:'验证码不能为空'
                    },
                    {
                       pattern: /^\d{6}$/,
                        message:'验证码格式错误' 
                    }
                ]
            },
            isCountDownShow:false,  //控制倒计时的显示和隐藏

        }
    },
  methods: {
    // 表单提交事件
  async onSubmit (values) {
          console.log('submit', values)
        //   获取表单数据
          const user = this.user
        //   表单验证
        this.$toast.loading({
         message: '登录中...', //禁用背景点击
            forbidClick: true,
           duration:0     //持续时间，默认是2000，如果为0 则持续展示
            });
        // 提交表单请求登录
          try {
              const res = await login(user)
              console.log(res);
        this.$toast.success('登录成功');

          } catch (err) {
              if (err.response && err.response.status === 400) {
           this.$toast.fail('手机号或验证码错误');
              } else {
            this.$toast.fail('登录失败，请稍后重试');
         }
          }
    //   根据请求响应结果处理后续操作
      },
    //   验证手机号
     async onSendSms() {
         try {
            await this.$refs.loginForm.validate('mobile')
          } catch (err) {
            return console.log(err);  
          }
          // 验证通过显示倒计时
         this.isCountDownShow=true
         // 发送请求验证码
         try {
             await sendSms(this.user.mobile)
            this.$toast('发送成功')
         } catch (err) {
            // 发送失败，关闭倒计时
             this.isCountDownShow = false
             if (err.response&&err.response.status === 429) {
                this.$toast('发送太频繁了，请稍微重试')
             } else {
                this.$toast('发送失败，请稍后重试')
            }
         }
    }
  }
}
</script>

<style lang="less" scoped>
.login-container{
.toutiao{
    font-size: 37px;
}
.send-sms-btn{
    height: 46px;
    line-height: 46px;
    background-color: #ededed;
    font-size: 22px;
    color: #ccc;
    border: none;
    border-radius: 23px;
}
.login-btn-wrap{
   padding: 53px 33px;
	.login-btn{
        background-color: #6db4fb;
        border: none;
        border-radius: 10px;
    }

}
}
</style>