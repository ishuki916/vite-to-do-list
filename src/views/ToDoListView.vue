<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { walk } from 'vue/compiler-sfc'
const router = useRouter()

const userName = ref('')
const myCookie = ref('')
const isValid = ref(false)
const toDoList = ref({})
const toDoSize = computed(() => toDoList.value.length)
const content = ref('')
onMounted(async () => {
  try {
    myCookie.value = document.cookie.replace(/(?:(?:^|.*;\s*)tkn\s*\=\s*([^;]*).*$)|^.*$/, '$1')
    const res = await axios.get('https://todolist-api.hexschool.io/users/checkout', {
      headers: {
        authorization: myCookie.value
      }
    })

    if (res.data.status === true) {
      userName.value = res.data.nickname
      const resTodo = await axios.get('https://todolist-api.hexschool.io/todos', {
        headers: {
          authorization: myCookie.value
        }
      })

      toDoList.value = resTodo.data.data
    }
  } catch (err) {
    loginPage()
  }
})

const signOut = async () => {
  try {
    const res = await axios.post(
      'https://todolist-api.hexschool.io/users/sign_out',
      {},
      {
        headers: {
          authorization: myCookie.value
        }
      }
    )

    if (res.data.status) {
      document.cookie = `tkn=; max-age=0;path=/; `
      userName.value = ''
      myCookie.value = ''
      loginPage()
    } else {
      alert('登出有錯誤')
    }
  } catch (err) {
    alert(err)
  }
}
const loginPage = () => {
  router.push('/login')
}

const insertTodo = async () => {
  try {
    const res = await axios.post(
      'https://todolist-api.hexschool.io/todos',
      { content: content.value },
      {
        headers: {
          authorization: myCookie.value
        }
      }
    )
    toDoList.value.push(res.data.newTodo)
  } catch (err) {
    console.log('insert err:', err.response.data.message)
  } finally {
    content.value = ''
  }
}
const toggleToDo = async (toDO) => {
  try {
    const res = await axios.patch(
      `https://todolist-api.hexschool.io/todos/${toDO.id}/toggle`,
      {},
      {
        headers: {
          authorization: myCookie.value
        }
      }
    )
  } catch (err) {
    console.log('toggle err', err)
  }
}

const deleteToDo = async (toDO) => {
  try {
    const res = await axios.delete(`https://todolist-api.hexschool.io/todos/${toDO.id}`, {
      headers: {
        authorization: myCookie.value
      }
    })
    toDoList.value.forEach((item, index) => {
      if (item.id === toDO.id) {
        toDoList.value.splice(index, 1)
      }
    })
  } catch (err) {
    console.log('delete err:', err.response.data.message)
  }
}
</script>
<template>
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm">
          <span>使用者：{{ userName }} </span>
        </li>
        <li><a @click="signOut">登出</a></li>
      </ul>
    </nav>
    <div class="conatiner todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="content" />
          <a @click="insertTodo">
            <i class="fa fa-plus"></i>
          </a>
        </div>
        <div class="todoList_list">
          <ul class="todoList_tab">
            <li><a href="#" class="active">全部</a></li>
            <li><a href="#">待完成</a></li>
            <li><a href="#">已完成</a></li>
          </ul>
          <div class="todoList_items">
            <ul class="todoList_item">
              <li v-for="toDo in toDoList" :key="toDo.id">
                <label class="todoList_label">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    :checked="toDo.status"
                    @change="toggleToDo(toDo)"
                  />
                  <span>{{ toDo.content }}</span>
                </label>
                <a @click="deleteToDo(toDo)">
                  <i class="fa fa-times"></i>
                </a>
              </li>
            </ul>
            <div class="todoList_statistics">
              <p>{{ toDoSize }} 個待完成項目</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped></style>
