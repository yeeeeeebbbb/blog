<template>
  <div v-if="isLogin">
  <div class="login-panel">
      <n-card title="管理后台登录">
          <n-form :rules="rules" :model="admin">
              <n-form-item path="account" label="账号">
                  <n-input v-model:value="admin.account" placeholder="请输入账号" />
              </n-form-item>
              <n-form-item path="password" label="密码">
                  <n-input v-model:value="admin.password" type="password" placeholder="请输入密码" />
              </n-form-item>
          </n-form>
          <template #footer>
              <n-checkbox v-model:checked="admin.rember" label="记住我" />
              <n-button @click="login">登录</n-button>
          </template>
      </n-card>
  </div></div>
  <div v-else>
    <n-card title="管理后台注册">
      <n-form :rules="rules" :model="newAdmin">
          <n-form-item path="account" label="账号">
              <n-input v-model:value="newAdmin.account" placeholder="请输入账号" />
          </n-form-item>
          <n-form-item path="password" label="密码">
              <n-input v-model:value="newAdmin.password" type="password" placeholder="请输入密码" />
          </n-form-item>
          <n-form-item path="confirmPassword" label="确认密码">
              <n-input v-model:value="newAdmin.confirmPassword" type="password" placeholder="请确认密码" />
          </n-form-item>
      </n-form>
      <template #footer>
          <n-button @click="register">注册</n-button>
      </template>
    </n-card>
  </div>
  <button @click="toggleForm">切换到{{ isLogin ? '注册' : '登录' }}</button>
</template>

<script setup>

import { ref, reactive, inject } from 'vue'
import { AdminStore } from '../stores/AdminStore'

import { useRouter, useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()

const message = inject("message")
const axios = inject("axios")
const adminStore = AdminStore()
const isLogin = ref(true); // 控制显示登录还是注册表单
const toggleForm = () => {
  isLogin.value = !isLogin.value;
};

// 新增注册所需的数据和规则
const newAdmin = reactive({
  account: '',
  password: '',
  confirmPassword: ''
});

const registerRules = {
  account: [
      { required: true, message: "请输入账号", trigger: "blur" },
      { min: 3, max: 12, message: "账号长度在 3 到 12 个字符", trigger: "blur" },
  ],
  password: [
      { required: true, message: "请输入密码", trigger: "blur" },
      { min: 6, max: 18, message: "密码长度在 6 到 18 个字符", trigger: "blur" },
  ],
  confirmPassword: [
    {
      validator: (rule, value, callback) => {
        if (value !== newAdmin.password) {
          callback(new Error('两次输入的密码不一致'));
        } else {
          callback();
        }
      },
      trigger: 'blur'
    }
  ],
};
/**验证表单规则 */
let rules = {
  account: [
      { required: true, message: "请输入账号", trigger: "blur" },
      { min: 3, max: 12, message: "账号长度在 3 到 12 个字符", trigger: "blur" },
  ],
  password: [
      { required: true, message: "请输入密码", trigger: "blur" },
      { min: 6, max: 18, message: "密码长度在 6 到 18 个字符", trigger: "blur" },
  ],
};

/**管理员登录数据 */
const admin = reactive({
  account: localStorage.getItem("account") || "",
  password: localStorage.getItem("password") || "",
  rember: localStorage.getItem("rember") == 1 || false
})

/**登录 */
const login = async () => {
  let result = await axios.post("/admin/login", {
      account: admin.account,
      password: admin.password
  });
  if (result.data.code == 200) {
      adminStore.token = result.data.data.token
      adminStore.account = result.data.data.account
      adminStore.id = result.data.data.id

      //把数据存储到localStorage
      if (admin.rember) {
          localStorage.setItem("account", admin.account)
          localStorage.setItem("password", admin.password)
          localStorage.setItem("rember", admin.rember ? 1 : 0)
      }
      router.push("/dashboard")
      message.info("登录成功")
  } else {
      message.error("登录失败")
  }

}
const register = async () => {
  try {
    // 这里假设您已经添加了相应的表单验证逻辑
    // 并且验证通过了
    const response = await axios.post('/admin/register', {
      account: newAdmin.account,
      password: newAdmin.password,
    });
    console.response.data.code
    if (response.data.code === 200) {
      // 注册成功，处理逻辑（例如: 跳转到登录页面或显示成功消息）
    } else {
      // 注册失败，显示错误消息
      message.error("注册失败: " + response.data.msg);
    }
  } catch (error) {
    console.error(error);
    message.error("注册过程中发生错误");
  }
};
</script>

<style lang="scss" scoped>
.login-panel {
  width: 500px;
  margin: 0 auto;
  margin-top: 130px;
}
</style>