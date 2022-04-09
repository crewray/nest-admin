<template>
  <el-dialog title="用户信息编辑" v-model="visible" top="10vh" width="500px" :before-close="handleClose" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form ref="userFormRef" :model="userForm" label-width="80px" :rules="userFormRules" v-loading="loading">
      <el-form-item label="帐号" prop="account">
        <el-input v-model="userForm.account" ></el-input>
      </el-form-item>
      <el-form-item label="头像" prop="avatar">
        <el-avatar :src='userForm.avatar' style="cursor: pointer;" @click="avatarClickEvent"></el-avatar>
      </el-form-item>
      <el-form-item label="手机号" prop="phoneNum">
        <el-input v-model.trim="userForm.phoneNum"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model.trim="userForm.email"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model.trim="userForm.password"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="confirmPassword">
        <el-input v-model.trim="userForm.confirmPassword"></el-input>
      </el-form-item>
      
    </el-form>

    <!-- 头像编辑 -->
    <avatar-cropper v-model="showAvatarCropper" :avatar-url="userForm.avatar" @change="uploadSuccess"></avatar-cropper>
    <template #footer>
      <el-button @click="handleClose">取消</el-button>
      <el-button type="primary" @click="updateOrCreate">确认</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch } from 'vue'
import { ElMessage } from 'element-plus'

import { UPDATE_MODEL_EVENT } from '@/common/contants'
import { validPhone, validEmail } from '@/utils/validate'

import { ICreateOrUpdateUser, getUserInfo as getUserInfoApi, updateUser, getUserRoleIds as getUserRoleIdsApi, createUser, ICreateUser } from '@/api/user'
import { RoleApiResult } from '@/api/role'

import AvatarCropper from './AvatarCropper.vue'

export default defineComponent({
  name: 'AddUser',
  components: { AvatarCropper },
  props: {
    modelValue: {
      type: Boolean,
      default: false
    },
    currId: {
      type: String,
      default: () => {
        return null
      }
    },
    allRoles: {
      type: Array as PropType<RoleApiResult[]>,
      default: () => []
    }
  },
  emits: [UPDATE_MODEL_EVENT, 'change'],
  setup (props, { emit }) {
    const userFormRef = ref()

    const loading = ref(false)
    const userFormDefault = {
      
      account: '',
      phoneNum: '',
      email: '',
      avatar: '',
      roleIds: [],
      password:'',
      confirmPassword:''
    }
    // 表单
    const userForm = ref<ICreateUser>({ ...userFormDefault })

    const handleClose = () => {
      emit(UPDATE_MODEL_EVENT, false)
    }

    const validPhoneNum = (rule: any, value: string, callback: any) => {
      if (validPhone(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的手机号'))
      }
    }

    const validEmailFn = (rule: any, value: string, callback: any) => {
      if (validEmail(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的邮箱'))
      }
    }

    const userFormRules = {
    //   avatar: [
    //     { required: true, message: '请上传头像', trigger: 'blur' }
    //   ],
      account: [
        { required: true, message: '请输入用户名', trigger: 'blur' }
      ],
      password: [
        { required: true, message: '请输入密码', trigger: 'blur' }
      ],
      confirmPassword: [
        { required: true, message: '请输入确认密码', trigger: 'blur' }
      ],
      phoneNum: [
        { required: true, message: '请输入手机号', trigger: 'blur' },
        { validator: validPhoneNum, trigger: 'blur' }
      ],
      email: [
        { required: true, message: '请输入邮箱', trigger: 'blur' },
        { validator: validEmailFn, trigger: 'blur' }
      ],
      roleIds: [
        { type: 'array', required: true, message: '请至少选择一个角色', trigger: 'blur' }
      ]
    }

    const getUserInfo = async (currId: string) => {
      loading.value = true
      const res = await getUserInfoApi(currId)
      loading.value = false
      if (res?.code === 200) {
        userForm.value = res.data as ICreateUser
      }
    }

    const getUserRoleIds = async (currId: string) => {
      const res = await getUserRoleIdsApi(currId)
      if (res?.code === 200) {
        userForm.value.roleIds = res.data as number[]
      }
    }

    const updateOrCreate = async () => {
      userFormRef.value.validate(async (valid: boolean) => {
        if (valid) {
          loading.value = true
          const res = await createUser(userForm.value)
          loading.value = false
          if (res?.code === 200) {
            ElMessage({ type: 'success', message: '更新成功' })
            emit('change')
            handleClose()
          } else {
            ElMessage({ type: 'error', message: res?.msg || '网络异常，请稍后重试！' })
          }
        }
      })
    }

    // 头像编辑
    const showAvatarCropper = ref<boolean>(false)
    const avatarClickEvent = () => {
      showAvatarCropper.value = !showAvatarCropper.value
    }

    const uploadSuccess = (url: string) => {
      userForm.value.avatar = url
    }

    // dialog
    const visible = ref<boolean>(false)
    watch(() => props.modelValue, (val: boolean) => {
      visible.value = val
      if (val) {
        userForm.value = { ...userFormDefault }
      }
      if (val && props.currId) {
        getUserInfo(props.currId)
        getUserRoleIds(props.currId)
      }
    })

    return {
      loading,
      visible,
      handleClose,
      userForm,
      userFormRef,
      userFormRules,
      updateOrCreate,
      showAvatarCropper,
      avatarClickEvent,
      uploadSuccess
    }
  }
})
</script>
