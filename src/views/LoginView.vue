<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

const router = useRouter()
const user = ref({
  email: '',
  password: ''
})

const login = async () => {
  try {
    const res = await axios.post('https://todolist-api.hexschool.io/users/sign_in', user.value)

    document.cookie = `tkn=${res.data.token}; expires=${res.data.exp};path=/; `
  } catch (err) {
    alert(err.response.data.message)
    user.value = {}
  }
}

const signUpPage = () => {
  router.push('/signUp')
}
</script>
<template>
  <div id="loginPage" class="bg-yellow">
    <div class="conatiner loginPage vhContainer">
      <div class="side">
        <a href="#"
          ><img
            class="logoImg"
            src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
            alt=""
        /></a>
        <img
          class="d-m-n"
          src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
          alt="workImg"
        />
      </div>
      <div>
        <form class="formControls" action="index.html">
          <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
          <label class="formControls_label" for="email">Email</label>
          <input
            class="formControls_input"
            type="text"
            id="email"
            name="email"
            v-model="user.email"
            placeholder="請輸入 email"
            required
          />
          <span>此欄位不可留空</span>
          <label class="formControls_label" for="pwd">密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            v-model="user.password"
            placeholder="請輸入密碼"
            required
          />
          <input class="formControls_btnSubmit" type="button" @click="login" value="登入" />
          <a class="formControls_btnLink" @click="signUpPage">註冊帳號</a>
        </form>
      </div>
    </div>
  </div>
</template>
