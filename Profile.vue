<template>
  <div class="profile-container">
    <!-- Header 区域：头像、用户名、编辑按钮 -->
    <el-card class="profile-header-card">
      <div class="profile-header">
        <div class="avatar-section" @click="triggerAvatarSelect">
          <img
            :src="avatarPreview || user?.avatar || defaultAvatar"
            class="avatar-image"
            alt="头像"
          />
          <input
            type="file"
            ref="avatarInput"
            style="display: none"
            accept="image/png, image/jpeg"
            @change="handleAvatarChange"
          />
        </div>
        <div class="user-info">
          <h2 class="username">{{ user?.username || '未登录用户' }}</h2>
          <el-button v-if="user" type="primary" @click="handleEdit">编辑资料</el-button>
        </div>
      </div>
    </el-card>

    <!-- 表单信息区域 -->
    <el-card class="profile-card" v-if="isEditing">
      <el-form
        ref="profileForm"
        :model="userForm"
        :rules="rules"
        label-width="100px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="userForm.username" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="userForm.email" />
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="userForm.phone" />
        </el-form-item>
        <el-form-item label="新密码" prop="newPassword">
          <el-input type="password" v-model="userForm.newPassword" placeholder="如不修改请留空" />
        </el-form-item>
        <el-form-item label="确认密码" prop="confirmPassword">
          <el-input type="password" v-model="userForm.confirmPassword" placeholder="如不修改请留空" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleSave">保存</el-button>
          <el-button @click="handleCancel">取消</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 阅读概况区域 -->
    <ProfileOverview />

    <!-- 最近阅读记录预览模块 -->
    <el-card class="recent-logs-card">
      <template #header>
        <div class="card-header">
          <span>最近阅读记录</span>
        </div>
      </template>
      <el-table :data="recentLogs" style="width: 100%">
        <el-table-column prop="bookTitle" label="书名" />
        <el-table-column prop="readingMinutes" label="时长（分钟）" />
        <el-table-column prop="readingDate" label="日期" />
      </el-table>
      <div v-if="recentLogs.length === 0" class="no-logs">暂无阅读记录</div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import defaultAvatar from '@/assets/default-avatar.png'
import ProfileOverview from './ProfileOverview.vue'

const store = useStore()
const router = useRouter()

const isEditing = ref(false)
const avatarInput = ref(null)
const avatarPreview = ref(null)
const profileForm = ref(null)

const user = computed(() => store.state.user || {})
const logs = computed(() => store.state.logs || [])

const recentLogs = computed(() => logs.value.slice(-3).reverse())

const userForm = reactive({
  username: '',
  email: '',
  phone: '',
  newPassword: '',
  confirmPassword: '',
  avatar: null
})

const rules = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入有效的邮箱地址', trigger: ['blur', 'change'] }
  ],
  phone: [{ required: true, message: '请输入手机号', trigger: 'blur' }],
  newPassword: [
    {
      validator: (rule, value, callback) => {
        if (value && value.length < 6) {
          callback(new Error('新密码不能少于 6 位'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ],
  confirmPassword: [
    {
      validator: (rule, value, callback) => {
        if (userForm.newPassword && value !== userForm.newPassword) {
          callback(new Error('两次输入的密码不一致'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ]
}

onMounted(() => {
  store.dispatch('fetchUserProfile')
})

const handleEdit = () => {
  

  isEditing.value = true
  Object.assign(userForm, {
    username: user.value.username,
    email: user.value.email,
    phone: user.value.phone,
    newPassword: '',
    confirmPassword: '',
    avatar: null
  })
}

const handleSave = () => {
  profileForm.value.validate(async (valid) => {
    if (!valid) return

    const formData = new FormData()
    formData.append('username', userForm.username)
    formData.append('email', userForm.email)
    formData.append('phone', userForm.phone)
    if (userForm.newPassword) formData.append('password', userForm.newPassword)
    if (userForm.avatar) formData.append('avatar', userForm.avatar)

    try {
      await store.dispatch('updateUserProfile', formData)
      isEditing.value = false
      avatarPreview.value = null
      ElMessage.success('更新成功')
    } catch (error) {
      ElMessage.error('更新失败')
    }
  })
}

const handleCancel = () => {
  isEditing.value = false
  avatarPreview.value = null
}

const triggerAvatarSelect = () => {
  avatarInput.value?.click()
}

const handleAvatarChange = (e) => {
  const file = e.target.files[0]
  if (!file) return
  const allowedTypes = ['image/jpeg', 'image/png']
  const maxSize = 2 * 1024 * 1024
  if (!allowedTypes.includes(file.type)) {
    ElMessage.warning('仅支持 JPG / PNG 格式')
    return
  }
  if (file.size > maxSize) {
    ElMessage.warning('头像不能大于 2MB')
    return
  }
  userForm.avatar = file
  avatarPreview.value = URL.createObjectURL(file)
}
</script>

<style scoped>
.profile-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.profile-header-card {
  background: #1e2a38;
  color: #e0f0ff;
  border-radius: 12px;
  padding: 20px;
}

.profile-header {
  display: flex;
  align-items: center;
  gap: 20px;
}

.avatar-section {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  overflow: hidden;
  cursor: pointer;
  border: 2px solid #6cf;
  box-shadow: 0 0 12px rgba(0, 255, 255, 0.2);
  position: relative;
}

.avatar-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
}

.user-info {
  flex: 1;
}

.username {
  margin: 0;
  font-size: 24px;
  font-weight: bold;
  background: linear-gradient(to right, #6cf, #f0f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.profile-card,
.recent-logs-card {
  background: #1e2a38;
  color: #e0f0ff;
  border-radius: 12px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  padding: 20px;
}

.no-logs {
  padding: 20px;
  text-align: center;
  color: #aaa;
}
</style>
