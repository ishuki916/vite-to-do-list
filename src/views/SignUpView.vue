<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
const router = useRouter()
const newUser = ref({
  email: '',
  password: '',
  nickname: ''
})
const checkPwd = ref('')
const signUp = async () => {
  if (newUser.value.password !== checkPwd.value) {
    alert('密碼確認欄位不一致')
    checkPwd.value = ''
    return
  }
  try {
    const res = await axios.post('https://todolist-api.hexschool.io/users/sign_up', newUser.value)
    loginPage()
  } catch (err) {
    console.log(err.response.data.message)
    alert(err.response.data.message)
    newUser.value = {}
    checkPwd.value = ''
  }
}
const loginPage = () => {
  router.push('/login')
}
</script>
<template>
  <div id="signUpPage" class="bg-yellow">
    <div class="conatiner signUpPage vhContainer">
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
          <h2 class="formControls_txt">註冊帳號</h2>
          <label class="formControls_label" for="email">Email</label>
          <input
            class="formControls_input"
            type="text"
            id="email"
            name="email"
            placeholder="請輸入 email"
            v-model="newUser.email"
            required
          />
          <label class="formControls_label" for="name">您的暱稱</label>
          <input
            class="formControls_input"
            type="text"
            name="name"
            id="name"
            placeholder="請輸入您的暱稱"
            v-model="newUser.nickname"
          />
          <label class="formControls_label" for="pwd">密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            placeholder="請輸入密碼"
            v-model="newUser.password"
            required
          />
          <label class="formControls_label" for="pwd">再次輸入密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="checkPwd"
            placeholder="請再次輸入密碼"
            v-model="checkPwd"
            required
          />
          <input class="formControls_btnSubmit" type="button" @click="signUp" value="註冊帳號" />
          <a class="formControls_btnLink" @click="loginPage">登入</a>
        </form>
      </div>
    </div>
  </div>
</template>
<style scoped></style>
