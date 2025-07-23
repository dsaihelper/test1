<template>
  <div class="login-container">
    <a-card title="用户登录" :bordered="false" class="login-card">
      <a-form layout="vertical" class="login-form">
        <a-form-item label="用户名">
          <a-input v-model:value="username" placeholder="请输入用户名" />
        </a-form-item>
        <a-form-item label="密码">
          <a-input-password v-model:value="password" placeholder="请输入密码" />
        </a-form-item>
        <a-form-item>
          <a-button type="primary" block @click="handleLogin">登录</a-button>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { message } from "ant-design-vue";

const username = ref("");
const password = ref("");
const router = useRouter();

const handleLogin = () => {
  // 简单登录验证
  if (username.value && password.value) {
    const userRole = username.value === "admin" ? "admin" : "user";
    localStorage.setItem("userRole", userRole);
    message.success("登录成功");
    router.push("/home");
  } else {
    message.error("请输入用户名和密码");
  }
};
</script>

<style lang="less" scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f5f5f5;

  .login-card {
    width: 400px;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);

    .login-form {
      margin-top: 20px;
    }
  }
}
</style>