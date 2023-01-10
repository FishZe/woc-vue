<template>
  <div class="index">
  <ElContainer direction="vertical" class="container">
    <Header :user-name=userName :is-login=isLogin @on-logout="onLogout"/>

    <Main v-if="isLogin"/>
    <Login v-else @on-login="onLogin" />

    <Footer/>

  </ElContainer>
</div>
</template>

<script setup>
import axios from "axios";
import Header from "./components/Header.vue";
import Login from "./components/Login.vue";
import Footer from "./components/Footer.vue";
import Main from "./components/Main.vue"
import {onMounted, ref} from "vue";
import {ElNotification} from "element-plus";

axios.defaults.baseURL = '/api/'

const isLogin = ref(false)
const userName = ref("No User")
const onLogout = () => {
  isLogin.value = false
}
const onLogin = (data) => {
  isLogin.value = true
  userName.value = data
}

onMounted(
() => {
      getLoginUser()
    }
)
function getLoginUser() {
  axios
      .post("/me", {})
      .then(response => {
            if (response.data.code === 200) {
              isLogin.value = true
              userName.value = response.data.data.name
            } else {
              isLogin.value = false
            }
          }
      )
      .catch(error => {
        console.log(error)
      });
}

</script>

<style scoped>

.index {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.container {
  height: 100%;
}

</style>