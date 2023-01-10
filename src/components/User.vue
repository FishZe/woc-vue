<template>
  <el-dialog
      v-model="show"
      :title=title
      width="30%"
      :before-close="handleClose"
  >
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
        <el-input v-model="ruleForm.pass" type="password" autocomplete="off" :disabled="disablePassword"/>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="ruleForm.email" type="email" autocomplete="off"/>
      </el-form-item>
      <el-form-item label="角色" prop="role">
        <el-select v-model="ruleForm.role" placeholder="请选择角色">
          <el-option label="游客" value="-1"/>
          <el-option label="用户" value="0"/>
          <el-option label="管理员" value="1"/>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm(Form)">确定</el-button>
        <el-button @click="resetForm(Form)">重填</el-button>
      </el-form-item>
    </el-form>
  </el-dialog>
</template>

<script setup>
import {ref, reactive, watch, onMounted} from 'vue'

const emit = defineEmits(['onPanelClose','onSubmit'])
const props = defineProps({
  'isShow': {
    type: Boolean
  },
  'title': {
    type: String
  },
  'panelType': {
    type: String,
  },
  'user': {
    type: Object
  }
})
const Form = ref(null), disablePassword = ref(false)
const ruleForm = reactive({
  id:0,
  name: '',
  pass: '',
  email: '',
  role: "请选择角色",
})
let show = props.isShow,  title = props.title, panelType = props.panelType

watch(
    () => [props.isShow, props.title,props.panelType,props.user],
    ([newIsShow, newTitle, newType,newUser], [o1, o2, o3, o4]) => {
      show = newIsShow
      title = newTitle
      panelType = newType
      ruleForm.id = newUser.id
      ruleForm.name = newUser.name
      ruleForm.pass = newUser.pass
      ruleForm.email = newUser.email
      ruleForm.role = newUser.role
      if (panelType==="edit"){
        disablePassword.value = true
      }
    },
    [{immediate: true}, {immediate: true},{immediate: true},{immediate: true}]
)

const handleClose = () => {
  Form.value.resetFields()
  emit('onPanelClose')
}

const validateName = (rule, value, callback) => {
  if (value === '' && panelType === "new") {
    callback(new Error('请输入用户名'))
  } else {
    callback()
  }
}

const validatePass = (rule, value, callback) => {
  if (value === '' && panelType === "new") {
    callback(new Error('请输入密码'))
  } else {
    callback()
  }
}

const validateEmail = (rule, value, callback) => {
  let reg = /^[0-9a-zA-Z_.-]+[@][0-9a-zA-Z_.-]+([0-9a-zA-Z_.-]+){1,2}$/;
  if(value === ''){
    callback()
  } else if(!reg.test(value)){
    callback(new Error('请输入正确的邮箱'))
  } else {
    callback()
  }
}

const validateRole = (rule , value, callback) => {
  if (value === "请选择角色" && panelType === "new") {
    callback(new Error('请选择角色'))
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
  email: [
    {validator: validateEmail, trigger: 'blur'}
  ],
  role : [
    {validator: validateRole, trigger: 'blur'}
  ]
})

const roleDict = {"请选择角色":-2,"-1":-1,"0":0,"1":1}
const submitForm = (formEl) => {
  if (!formEl) return
  formEl.validate((valid) => {
    if (valid) {
      let data = {
        type: panelType,
        id: ruleForm.id,
        name: ruleForm.name,
        pass: ruleForm.pass,
        email: ruleForm.email,
        role: roleDict[ruleForm.role]
      }
      emit('onSubmit', data)
    } else {
      return false
    }
  })
}
const resetForm = (formEl) => {
  if (!formEl) return
  formEl.resetFields()
}

</script>

<style scoped>

</style>