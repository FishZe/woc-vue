<template>
  <el-header height="40px" style="border-bottom: 1px solid #4C4D4F;">
    <el-dropdown style="float: right">
      <el-button type="primary" style="margin-top: 3px">
        <el-icon size="22" style="margin-right: 3px"><User /></el-icon>
        <span v-if=props.isLogin>{{props.userName}}</span>
        <span v-else >未登录</span>
        <el-icon v-if=props.isLogin style="margin-left: 3px"><arrow-down /></el-icon>
      </el-button>
      <template #dropdown>
        <el-dropdown-menu v-if=props.isLogin >
          <el-dropdown-item @click="clickLogout">退出</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </el-header>
</template>

<script setup>
import axios from "axios";
import {ElNotification} from "element-plus";
const emit = defineEmits(['onLogout'])
const props = defineProps(['isLogin', 'userName'])

axios.defaults.baseURL = '/api/'

const clickLogout = () => {
  axios
      .post("/logout", {})
      .then(response => {
            if (response.data.code === 200) {
              ElNotification({
                title: '退出成功',
                message: '拜拜: ' + props.userName,
                type: 'success',
              })
              emit('onLogout')
            } else {
              ElNotification({
                title: '登录失败',
                message: response.data.msg,
                type: 'error',
              })
            }
          }
      )
      .catch(error => {
        console.log(error)
      });
}

</script>

<style scoped>

</style>