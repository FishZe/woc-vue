<template>
  <el-main>
    <el-input-number v-model="pageSize" :step="5" :min="5"/>
    <el-button type="primary" style="float: right;margin-bottom: 20px;margin-left: 20px" @click="mkNewUserBtn">新建</el-button>
    <el-button type="primary" style="float: right;margin-bottom: 20px;" @click="searchUserBtn" v-if="!isSearching">搜索</el-button>
    <el-button type="primary" style="float: right;margin-bottom: 20px;" @click="cancelUserBtn" v-else>取消</el-button>
    <el-table :data="tableData" border stripe width="50%" ref="allUsers" max-height="600">
      <el-table-column prop="Id" label="编号" width="80"/>
      <el-table-column prop="UserName" label="用户名" />
      <el-table-column prop="Password" label="密码" />
      <el-table-column prop="Email" label="邮箱" />
      <el-table-column prop="Role" label="角色" width="120"/>
      <el-table-column fixed="right" label="操作" width="150">
        <template  #default="scope">
          <el-button link type="primary" size="small" @click="userEdit(scope.row)">编辑</el-button>
          <el-button link type="primary" size="small" @click="userDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="margin-top: 20px">
    <ElPagination
        v-if="!isSearching"
        :pager-count="11"
        :page-size="pageSize"
        layout="prev, pager, next"
        :total="pageSum"
        :current-page="nowPage"
        background
        style="display: flex;align-content: center;justify-content: center;"
        @current-change="onPageChange"
    />
    </div>
    <User
      :title=userPanel.title
      :isShow=userPanel.show
      :panel-type=userPanel.panelType
      :user=userPanel.user
      @on-panel-close="userPanelClose"
      @on-submit="userPanelSubmit"
    />
  </el-main>
</template>

<script setup>
import axios from "axios";
import md5 from 'js-md5';
import User from "./User.vue";
import {h, onMounted, reactive, ref, watch} from "vue";
import {ElMessageBox, ElNotification} from "element-plus";

axios.defaults.baseURL = '/api/'

const allUsers = ref(null),pageSum = ref(0),isSearching = ref(false), nowPage = ref(1), pageSize = ref(15)

let userPanel = reactive({
  show: false,
  title: "新建用户",
  panelType: "new",
  user: reactive({
    id: 0,
    name: "",
    pass: "",
    email: "",
    role: "请选择角色",
  }),
})

const mkNewUserBtn = () => {
  userPanel.show = true
  userPanel.panelType = "new"
  userPanel.title = "新建用户"
  userPanel.user = reactive({
    id: 0,
    name: "",
    pass: "",
    email: "",
    role: "请选择角色",
  })
}

const searchUserBtn = () => {
  userPanel.show = true
  userPanel.panelType = "search"
  userPanel.title = "搜索用户"
  userPanel.user = reactive({
    id: 0,
    name: "",
    pass: "",
    email: "",
    role: "请选择角色",
  })
}

const cancelUserBtn = () => {
  isSearching.value = false
  getSomeUser(nowPage.value)
}

watch(
    () => pageSize.value,
    (newPageSize) => {
      if(newPageSize < 5) {
        ElNotification({
          title: '警告',
          message: '每页显示的用户数不能小于5',
          type: 'error',
        })
      }
      getSomeUser(1)
    }
)

const userEdit = (e) => {
  console.log(e.Id)
  userPanel.show = true
  userPanel.panelType = "edit"
  userPanel.title = "编辑用户"
  userPanel.user = reactive({
    id: e.Id,
    name: e.UserName,
    pass: e.Password,
    email: e.Email,
    role: e.Role,
  })
}

const userDelete = (e) => {
  axios
      .post("/admin/delete", {"id":e.Id})
      .then(response => {
            if (response.data.code === 200) {
              ElNotification({
                title: '删除成功',
                message: '删除了用户: ' + e.UserName,
                type: 'success',
              })
            } else {
              ElNotification({
                title: '删除失败',
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
const userPanelClose = () => {
  userPanel.show = false
}

const onPageChange = (e) => {
  nowPage.value = e
  console.log(nowPage)
  getSomeUser(e)
}
const userPanelSubmit = (data) => {
  let nowUser  = {
    "id": data.id,
    "name": data.name,
    "password": md5(data.pass),
    "email": data.email,
    "role": data.role,
  }
  userPanel.show = false
  switch (data.type) {
    case "new":
      insertNewUser(nowUser)
      getSomeUser(nowPage.value)
      break
    case "edit":
      editUser(nowUser)
      getSomeUser(nowPage.value)
      break
    case "search":
      if(data.id === 0 && data.name === "" && data.email === "" && data.role === -2 && data.pass === "") {
        ElNotification({
          title: '警告',
          message: '搜索条件不能为空',
          type: 'error',
        })
      } else {
        if(data.pass === "") {
          nowUser.password = ""
        }
        searchUser(nowUser)
        isSearching.value = true
      }
      break
    default:
      break
  }
}

const reverseRoleDict = {
  "-1": "游客",
  "0": "管理员",
  "1": "超级管理员",
}

function searchUser(nowUser) {
  axios
      .post("/admin/search", nowUser)
      .then(response => {
            if (response.data.code === 200) {
              tableData.value = []
              for (let i = 0; i < response.data.data.length; i++) {
                let now = {
                  Id: response.data.data[i].id,
                  UserName: response.data.data[i].name,
                  Password: response.data.data[i].password,
                  Email: response.data.data[i].email,
                  Role: reverseRoleDict[response.data.data[i].role],
                }
                tableData.value.push(now)
              }
            } else {
              ElNotification({
                title: '添加失败',
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

function insertNewUser(nowUser) {
  axios
      .post("/admin/new", nowUser)
      .then(response => {
            if (response.data.code === 200) {
              ElNotification({
                title: '添加成功',
                message: '用户添加成功: '+ nowUser.name,
                type: 'success',
              })
            } else {
              ElNotification({
                title: '添加失败',
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

function editUser(nowUser) {
  axios
      .post("/admin/update", nowUser)
      .then(response => {
            if (response.data.code === 200) {
              ElNotification({
                title: '编辑成功',
                message: '用户修改成功: '+ nowUser.name,
                type: 'success',
              })
            } else {
              ElNotification({
                title: '修改失败',
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

function getSomeUser(page){
  axios
      .post("/admin/get", {"from":(page - 1) * pageSize.value,"sum":pageSize.value})
      .then(response => {
            if (response.data.code === 200) {
              pageSum.value = response.data.data.all
              tableData.value = []
              for (let i = 0; i < response.data.data.user.length; i++) {
                let now = {
                  Id: response.data.data.user[i].id,
                  UserName: response.data.data.user[i].name,
                  Password: response.data.data.user[i].password,
                  Email: response.data.data.user[i].email,
                  Role: reverseRoleDict[response.data.data.user[i].role],
                }
                tableData.value.push(now)
              }
            } else {
              ElNotification({
                title: '获取用户失败',
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

const tableData = ref()

onMounted(
    () => {
       getSomeUser(1)
    }
)


</script>
<style scoped>
</style>