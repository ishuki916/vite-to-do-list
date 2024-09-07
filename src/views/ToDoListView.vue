<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { walk } from 'vue/compiler-sfc'
import AllToDoView from './AllToDoView.vue'
const router = useRouter()

const userName = ref('')
const myCookie = ref('')
const isValid = ref(false)
const toDoList = ref({})
const toDoSize = ref(0)
const content = ref('')
const activeIndex = ref(0)
const unFinishedList = ref([])
const finishedList = ref([])
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
      toDoList.value.forEach((item) => {
        if (!item.status) {
          toDoSize.value++
        }
      })
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
    toDoSize.value++
    unFinishedList.value.push(res.data.newTodo)
  } catch (err) {
    console.log('insert err:', err.response.data.message)
  } finally {
    content.value = ''
  }
}
const toggleToDo = async (toDO) => {
  try {
    toDO.status = !toDO.status
    if (toDO.status) {
      toDoSize.value--
    } else {
      toDoSize.value++
    }
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
    if (!toDO.status) {
      toDoSize.value--
    }
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

const setActiveIndex = (index) => {
  activeIndex.value = index
  finishedList.value = []
  unFinishedList.value = []
}

const handleUnFinished = () => {
  setActiveIndex(1)
  toDoList.value.forEach((item) => {
    if (!item.status) {
      unFinishedList.value.push(item)
    }
  })
}

const handleFinished = () => {
  setActiveIndex(2)
  toDoList.value.forEach((item) => {
    if (item.status) {
      finishedList.value.push(item)
    }
  })
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
            <li><a @click="setActiveIndex(0)" :class="{ active: activeIndex === 0 }">全部</a></li>
            <li><a @click="handleUnFinished" :class="{ active: activeIndex === 1 }">待完成</a></li>
            <li><a @click="handleFinished" :class="{ active: activeIndex === 2 }">已完成</a></li>
          </ul>
          <div class="todoList_items">
            <ul class="todoList_item">
              <li v-if="toDoList.length === 0">目前尚無待辦事項</li>
              <div v-if="activeIndex === 0">
                <AllToDoView
                  v-for="toDo in toDoList"
                  :key="toDo.id"
                  :todo="toDo"
                  @delete="deleteToDo(toDo)"
                  @toggle="toggleToDo(toDo)"
                />
                <div class="todoList_statistics">
                  <p>{{ toDoSize }} 個待完成項目</p>
                </div>
              </div>
              <div v-if="activeIndex === 1">
                <li v-for="unf in unFinishedList" :key="unf.id">
                  <span>{{ unf.content }}</span>
                </li>
              </div>
              <div v-if="activeIndex === 2">
                <li v-for="fi in finishedList" :key="fi.id">
                  <span>{{ fi.content }}</span>
                </li>
              </div>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped></style>
