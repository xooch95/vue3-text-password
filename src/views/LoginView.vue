<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router';
import TextPassword from '../components/TextPassword.vue';

const router = useRouter();
const loading = ref(false)
const loginForm = ref({
  username: "",
  password: ""
});

const loginRules = {
  username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
  password: [{ required: true, trigger: "blur", message: "请输入您的密码" }]
};

const handleLogin = () => {
  loading.value = true
  setTimeout(() => {
    router.push({ path: '/main' });
    loading.value = false;
  }, 1000)
  
}

</script>

<template>

  <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form" style="width: 500px">
    <h2>禁止浏览器弹出保存密码弹窗</h2>
    <el-form-item prop="username">
      <el-input
        v-model="loginForm.username"
        type="text"
        size="large"
        auto-complete="off"
        placeholder="账号"
      />
    </el-form-item>
    <el-form-item prop="password">
      <text-password 
        v-model="loginForm.password" 
        size="large"
        @enter="handleLogin"
        :auto-complete="false"
        placeholder="密码" />
    </el-form-item>

    <el-form-item style="width:100%;">
        <el-button
          :loading="loading"
          size="large"
          type="primary"
          style="width:100%;"
          @click.prevent="handleLogin"
        >
          <span v-if="!loading">登 录</span>
          <span v-else>登 录 中...</span>
        </el-button>
      </el-form-item>
  </el-form>
</template>

<style>
.login-form {
  text-align: center;
  margin: 0 auto;
  margin-top: 10rem;
}
</style>