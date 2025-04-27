<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const form = reactive({
  name: '',
  password: '',
  mobile: '',
  verification_code: '',
});
const activeName = ref('first');
const qrCode = ref('');
const message = ref('请使用 Bilibili 扫描二维码进行登录。');
const oauthKey = ref('');

let timer;
let countdown = ref(60);
let isCounting = ref(false);

const POLLING_LIMIT = 29000; // 3分钟限制 (3 * 60 * 1000)

function onSubmit() {
  router.push('/index');
}

function loadQRCode() {
  fetch('http://localhost:8888/v1/loginqr')
    .then(response => response.json())
    .then(data => {
      qrCode.value = data.qr_code;
      oauthKey.value = data.oauth_key;
      message.value = "请使用 Bilibili 扫描二维码进行登录。"; // 重置消息
      checkQRStatus();

      if (timer) {
        clearTimeout(timer);
      }
      timer = setTimeout(stopPolling, POLLING_LIMIT);
    })
    .catch(error => {
      console.error("Error fetching QR code:", error);
    });
}



function register() {
  // Constructing the payload to send to the server
  const payload = {
    name: form.name,
    mobile: form.mobile,
    password: form.password,
    verification_code: form.verification_code,
  };
  console.log("注册请求 payload:", payload); // 打印请求的 payload

  fetch('http://127.0.0.1:8888/v1/register', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(payload),
  })
    .then(response => response.json())
    .then(data => {
        console.log("注册响应数据:", data); // 打印返回的数据
        if (data.user_id) {
        // 如果返回了 user_id，则说明注册成功
        message.value = "注册成功，正在跳转...";
        
        // 你可以在这里保存 token 或者其他用户信息
        localStorage.setItem('access_token', data.token.access_token);
        localStorage.setItem('refresh_token', data.refresh_token.access_token);
        
        // 跳转到主页或其他页面
        router.push('/index');
      } else {
        message.value = "注册失败：" + (data.message || "请重试。");
      }
    })
    .catch(error => {
      console.error("Error during registration:", error);
      message.value = "注册时出错，请稍后再试。";
    });
}



function checkQRStatus() {
  fetch('http://localhost:8888/v1/qrstatus?oauth_key=' + oauthKey.value)
    .then(response => response.json())
    .then(jsonData => {
      if (jsonData.status === "success") {
        console.log("登录成功，跳转中...");
        router.push('/index');
      } else if (jsonData.status === "expired") {
        message.value = "二维码已过期，请点击下方按钮刷新二维码。";
      } else {
        console.log("等待二维码扫描...");
        checkQRStatus();
      }
    })
    .catch(error => {
      console.error("Error fetching QR status:", error);
    });
}

function stopPolling() {
  if (timer) {
    clearTimeout(timer);
  }
  message.value = "二维码已过期，请点击下方按钮刷新二维码。";
}

function sendVerificationCode() {
  if (!form.mobile) {
    message.value = "请输入手机号";
    return;
  }

  fetch('http://127.0.0.1:8888/v1/verification', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ mobile: form.mobile }),
  })
    .then(response => response.json())
    .then(data => {
        message.value = "验证码已发送，请查看您的手机。";
        startCountdown();
    })
    .catch(error => {
      console.error("Error sending verification code:", error);
    });
}

function startCountdown() {
  isCounting.value = true;
  countdown.value = 60;
  const interval = setInterval(() => {
    if (countdown.value > 0) {
      countdown.value--;
    } else {
      clearInterval(interval);
      isCounting.value = false;
    }
  }, 1000);
}

onMounted(() => {
  loadQRCode();
});
</script>

<template>
<div class="login">
    <el-form :model="form" label-width="120px">
        <div class="login-img">
            <img src="@/assets/image/v2.png" alt="">
        </div>
        <div class="login-item">
            <div class="loginleft">
                <p class="loginleft-title">打开 Bilibili App</p>
                <div class="loginleft-erm">
                    <img :src="qrCode" alt="QR Code" width="256" />
                </div>
                <p>{{ message }}</p>
                <el-button v-if="message.includes('二维码已过期')" @click="loadQRCode">刷新二维码</el-button>
                <ul>
                    <li><el-button round>下载Beyond</el-button></li>
                    <li><el-button round>机构号</el-button></li>
                    <li><el-button round>无障碍模式</el-button></li>
                </ul>
            </div>
            <div class="loginright">
                <el-tabs v-model="activeName" class="demo-tabs">
                    <el-tab-pane label="账号登录" name="first">
                        <el-input v-model="form.name" placeholder="请输入手机/用户名" />
                        <el-input v-model="form.password" type="password" placeholder="请输入密码" />
                        <el-button type="primary" @click="onSubmit">登录</el-button>
                        <div class="otherwise">
                            <div class="otherwise-title">其他方式登录</div>
                            <ul>
                                <li>
                                    <svg class="icon-font">
                                    <use xlink:href="#icon-weixin"></use></svg>
                                </li>
                                <li>
                                    <svg class="icon-font">
                                    <use xlink:href="#icon-QQ"></use></svg>
                                </li>
                                <li>
                                    <svg class="icon-font">
                                    <use xlink:href="#icon-shejiaotubiao-06"></use></svg>
                                </li>
                            </ul>
                        </div>
                    </el-tab-pane>
                    <el-tab-pane label="注册" name="second">
                         <el-input v-model="form.mobile" placeholder="请输入手机" />
                         <div class="code-btn">
                            <el-input v-model="form.verification_code" placeholder="请输入验证码" />
                            <el-link 
                              type="primary" 
                              @click="sendVerificationCode" 
                              :disabled="isCounting"
                            >
                              {{ isCounting ? `${countdown}秒后重发` : '获取验证码' }}
                            </el-link>
                         </div>
                         <el-input v-model="form.name"  placeholder="请输入用户名" /> 
                        <el-input v-model="form.password" type="password" placeholder="请输入密码" /> 
                        <el-button type="primary" @click="register">注册</el-button>
                        <!-- <p class="agreement">注册即代表同意用户协议</p> -->
                    </el-tab-pane>
                </el-tabs>
            </div>
        </div>
    </el-form>
</div>
</template>

<style scoped>
 
</style>
