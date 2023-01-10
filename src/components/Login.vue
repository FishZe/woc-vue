<template>
  <el-main>
    <el-form
        ref="Form"
        :model="ruleForm"
        status-icon
        :rules="rules"
        label-width="60px"
        class="login"
    >
      <el-form-item label="用户名" prop="name">
        <el-input v-model="ruleForm.name"/>
      </el-form-item>
      <el-form-item label="密码" prop="pass">
        <el-input v-model="ruleForm.pass" type="password" autocomplete="off"/>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="submitForm(Form)">登录</el-button>
        <el-button @click="resetForm(Form)">重填</el-button>
      </el-form-item>
    </el-form>
  </el-main>
</template>

<script setup>
import axios from "axios";
import md5 from 'js-md5';
import {h, reactive, ref} from "vue";
import FormInstance, {ElMessageBox, ElNotification} from 'element-plus'

axios.defaults.baseURL = '/api/'

const emit = defineEmits(['onLogin'])
const Form = ref < FormInstance > ({})
const ruleForm = reactive({
  name: '',
  pass: '',
})

const validateName = (rule, value, callback) => {
  if (value === '') {
    callback(new Error('请输入用户名'))
  } else {
    callback()
  }
}

const validatePass = (rule, value, callback) => {
  if (value === '') {
    callback(new Error('请输入密码'))
  } else {
    callback()
  }
}

const rules = reactive({
  name: [
    {validator: validateName, trigger: 'blur'}
  ],
  pass: [
    {validator: validatePass, trigger: 'blur'}
  ],
})
const resetForm = (formEl) => {
  if (!formEl) return
  formEl.resetFields()
}

const submitForm = (formEl) => {
  if (!formEl) return
  formEl.validate((valid) => {
    if (valid) {
      axios
          .post("/login", {"name": ruleForm.name, "password": md5(ruleForm.pass)})
          .then(response => {
                if (response.data.code === 200) {
                  ElNotification({
                    title: '登陆成功',
                    message: h('i', {style: 'color: teal'}, '欢迎回来' + ruleForm.name),
                    type: 'success',
                  })
                  emit('onLogin',ruleForm.name)
                } else {
                  ElMessageBox.confirm(response.data.msg).catch(() => {
                  })
                }
              }
          )
          .catch(error => {
            console.log(error)
          });
    } else {
      console.log('error submit!')
      return false
    }
  })
}

</script>

<style scoped>

.login {
  width: 300px;
  margin: 200px auto 200px;
}

</style>