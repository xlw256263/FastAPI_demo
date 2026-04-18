<template>
  <div class="profile-container">
    <el-card class="profile-card" shadow="hover">
      <template #header>
        <div class="card-header">
          <span class="header-title">👤 个人中心</span>
        </div>
      </template>

      <div class="profile-content">
        <div class="profile-sidebar">
          <div class="avatar-wrapper">
            <el-avatar :size="150" :src="displayAvatar" class="user-avatar" @click="triggerUpload">
              <el-icon :size="80"><User /></el-icon>
            </el-avatar>
            <el-upload
              ref="uploadRef"
              :auto-upload="false"
              :show-file-list="false"
              :on-change="handleAvatarChange"
              accept="image/jpeg,image/png,image/gif,image/webp"
              class="avatar-uploader"
              style="display: none;"
            >
              <el-button type="primary" size="small" circle class="upload-btn">
                <el-icon><Camera /></el-icon>
              </el-button>
            </el-upload>
          </div>

          <div class="user-info-brief">
            <h3 class="username">{{ userStore.userInfo?.username }}</h3>
            <p class="user-email">{{ userStore.userInfo?.email }}</p>
            <el-tag v-if="userStore.userInfo?.bio" size="small" type="info" class="bio-tag">
              {{ userStore.userInfo.bio.length > 20 ? userStore.userInfo.bio.substring(0, 20) + '...' : userStore.userInfo.bio }}
            </el-tag>
          </div>

          <el-divider />

          <el-menu :default-active="activeMenu" @select="handleMenuSelect" class="profile-menu">
            <el-menu-item index="basic">
              <el-icon><User /></el-icon>
              <span>基本信息</span>
            </el-menu-item>
            <el-menu-item index="password">
              <el-icon><Lock /></el-icon>
              <span>修改密码</span>
            </el-menu-item>
            <el-menu-item index="security">
              <el-icon><Key /></el-icon>
              <span>账号安全</span>
            </el-menu-item>
          </el-menu>
        </div>

        <div class="profile-main">
          <div v-show="activeMenu === 'basic'" class="content-panel">
            <div class="panel-header">
              <h3>基本信息</h3>
              <el-button
                v-if="!isEditing"
                type="primary"
                plain
                @click="startEdit"
              >
                <el-icon><Edit /></el-icon>
                编辑资料
              </el-button>
              <div v-else class="edit-actions">
                <el-button @click="cancelEdit">取消</el-button>
                <el-button type="primary" @click="handleSaveBasic" :loading="saving">
                  <el-icon><Check /></el-icon>
                  保存
                </el-button>
              </div>
            </div>

            <div v-if="!isEditing" class="info-display">
              <el-descriptions :column="1" border class="info-descriptions">
                <el-descriptions-item label="用户名">
                  <span class="info-value">{{ displayForm.username }}</span>
                </el-descriptions-item>
                <el-descriptions-item label="邮箱">
                  <span class="info-value">{{ displayForm.email }}</span>
                </el-descriptions-item>
                <el-descriptions-item label="个人简介">
                  <span class="info-value bio-text">{{ displayForm.bio || '暂无简介' }}</span>
                </el-descriptions-item>
              </el-descriptions>
            </div>

            <el-form
              v-else
              ref="basicFormRef"
              :model="editForm"
              :rules="basicRules"
              label-width="100px"
              class="edit-form"
            >
              <el-form-item label="用户名" prop="username">
                <el-input v-model="editForm.username" placeholder="请输入用户名" />
              </el-form-item>

              <el-form-item label="邮箱" prop="email">
                <el-input v-model="editForm.email" placeholder="请输入邮箱" />
              </el-form-item>

              <el-form-item label="个人简介" prop="bio">
                <el-input
                  v-model="editForm.bio"
                  type="textarea"
                  :rows="4"
                  placeholder="介绍一下自己吧..."
                  maxlength="500"
                  show-word-limit
                />
              </el-form-item>
            </el-form>
          </div>

          <div v-show="activeMenu === 'password'" class="content-panel">
            <div class="panel-header">
              <h3>修改密码</h3>
            </div>

            <el-form
              ref="passwordFormRef"
              :model="passwordForm"
              :rules="passwordRules"
              label-width="100px"
              class="password-form"
            >
              <el-form-item label="原密码" prop="old_password">
                <el-input
                  v-model="passwordForm.old_password"
                  type="password"
                  placeholder="请输入原密码"
                  show-password
                  clearable
                />
              </el-form-item>

              <el-form-item label="新密码" prop="new_password">
                <el-input
                  v-model="passwordForm.new_password"
                  type="password"
                  placeholder="请输入新密码（6-100位）"
                  show-password
                  clearable
                />
              </el-form-item>

              <el-form-item label="确认密码" prop="confirm_password">
                <el-input
                  v-model="passwordForm.confirm_password"
                  type="password"
                  placeholder="请再次输入新密码"
                  show-password
                  clearable
                />
              </el-form-item>

              <el-form-item>
                <el-button type="primary" @click="handleChangePassword" :loading="changingPassword">
                  确认修改
                </el-button>
                <el-button @click="resetPasswordForm">重置</el-button>
              </el-form-item>
            </el-form>
          </div>

          <div v-show="activeMenu === 'security'" class="content-panel">
            <div class="panel-header">
              <h3>账号安全</h3>
            </div>

            <el-descriptions :column="1" border class="security-descriptions">
              <el-descriptions-item label="注册时间">
                <el-icon class="desc-icon"><Calendar /></el-icon>
                {{ formatDate(userStore.userInfo?.created_at) }}
              </el-descriptions-item>
              <el-descriptions-item label="账号状态">
                <el-icon class="desc-icon"><CircleCheck /></el-icon>
                <el-tag :type="userStore.userInfo?.is_active === 1 ? 'success' : 'danger'" size="small">
                  {{ userStore.userInfo?.is_active === 1 ? '正常' : '已禁用' }}
                </el-tag>
              </el-descriptions-item>
              <el-descriptions-item label="用户角色">
                <el-icon class="desc-icon"><Star /></el-icon>
                <el-tag :type="userStore.userInfo?.role === 'admin' ? 'warning' : 'info'" size="small">
                  {{ userStore.userInfo?.role === 'admin' ? '管理员' : '普通用户' }}
                </el-tag>
              </el-descriptions-item>
            </el-descriptions>

            <el-alert
              title="危险操作"
              type="error"
              :closable="false"
              show-icon
              class="danger-alert"
            >
              <template #default>
                <div class="danger-content">
                  <p>注销账号后，您的所有数据将被清空且无法恢复</p>
                  <el-popconfirm
                    title="确定要注销账号吗？此操作不可恢复！"
                    confirm-button-text="确定注销"
                    cancel-button-text="取消"
                    @confirm="handleDeleteAccount"
                  >
                    <template #reference>
                      <el-button type="danger" size="small">注销账号</el-button>
                    </template>
                  </el-popconfirm>
                </div>
              </template>
            </el-alert>
          </div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import { ElMessage, type FormInstance, type FormRules, type UploadFile } from 'element-plus'
import { User, Camera, Edit, Check, Lock, Key, Calendar, CircleCheck, Star } from '@element-plus/icons-vue'
import { useUserStore } from '@/stores/user'
import { updateProfile, changePassword, uploadAvatar, deleteUser } from '@/api/user'
import type { UserProfileUpdate, PasswordChange } from '@/types'

const userStore = useUserStore()

const activeMenu = ref('basic')
const isEditing = ref(false)
const saving = ref(false)
const changingPassword = ref(false)
const uploadRef = ref()

const basicFormRef = ref<FormInstance>()
const passwordFormRef = ref<FormInstance>()

const displayForm = reactive({
  username: '',
  email: '',
  bio: ''
})

const editForm = reactive({
  username: '',
  email: '',
  bio: ''
})

const passwordForm = reactive({
  old_password: '',
  new_password: '',
  confirm_password: ''
})

const validateConfirmPassword = (rule: any, value: string, callback: any) => {
  if (value !== passwordForm.new_password) {
    callback(new Error('两次输入的密码不一致'))
  } else {
    callback()
  }
}

const basicRules: FormRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 2, max: 50, message: '用户名长度为 2-50 个字符', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
  ],
  bio: [
    { max: 500, message: '个人简介不能超过 500 个字符', trigger: 'blur' }
  ]
}

const passwordRules: FormRules = {
  old_password: [
    { required: true, message: '请输入原密码', trigger: 'blur' }
  ],
  new_password: [
    { required: true, message: '请输入新密码', trigger: 'blur' },
    { min: 6, max: 100, message: '密码长度为 6-100 个字符', trigger: 'blur' }
  ],
  confirm_password: [
    { required: true, message: '请再次输入新密码', trigger: 'blur' },
    { validator: validateConfirmPassword, trigger: 'blur' }
  ]
}

const displayAvatar = computed(() => {
  const avatar = userStore.userInfo?.avatar
  if (!avatar) return ''

  // 如果已经是完整URL，直接返回
  if (avatar.startsWith('http')) return avatar

  // 否则拼接基础URL（去掉/api/v1）
  const baseUrl = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api/v1'
  const baseWithoutApi = baseUrl.replace('/api/v1', '')
  return `${baseWithoutApi}${avatar}`
})

const loadUserInfo = () => {
  if (userStore.userInfo) {
    displayForm.username = userStore.userInfo.username
    displayForm.email = userStore.userInfo.email
    displayForm.bio = userStore.userInfo.bio || ''

    editForm.username = userStore.userInfo.username
    editForm.email = userStore.userInfo.email
    editForm.bio = userStore.userInfo.bio || ''
  }
}

const handleMenuSelect = (index: string) => {
  activeMenu.value = index
}

const startEdit = () => {
  editForm.username = displayForm.username
  editForm.email = displayForm.email
  editForm.bio = displayForm.bio
  isEditing.value = true
}

const cancelEdit = () => {
  isEditing.value = false
}

const handleSaveBasic = async () => {
  if (!basicFormRef.value) return

  try {
    await basicFormRef.value.validate()
    saving.value = true

    const data: UserProfileUpdate = {
      username: editForm.username,
      email: editForm.email,
      bio: editForm.bio
    }

    const response = await updateProfile(data)
    userStore.setUserInfo(response)

    displayForm.username = editForm.username
    displayForm.email = editForm.email
    displayForm.bio = editForm.bio

    isEditing.value = false
    ElMessage.success('保存成功')
  } catch (error: any) {
    if (error.response?.status === 409) {
      ElMessage.error(error.response.data.detail || '用户名或邮箱已被使用')
    } else {
      ElMessage.error('保存失败，请稍后重试')
    }
  } finally {
    saving.value = false
  }
}

const handleAvatarChange = async (file: UploadFile) => {
  if (!file.raw) return

  const isImage = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'].includes(file.raw.type)
  const isLt5M = file.raw.size / 1024 / 1024 < 5

  if (!isImage) {
    ElMessage.error('只支持 JPG、PNG、GIF、WEBP 格式的图片')
    return
  }

  if (!isLt5M) {
    ElMessage.error('图片大小不能超过 5MB')
    return
  }

  try {
    saving.value = true
    const response = await uploadAvatar(file.raw)

    const updatedUser = { ...userStore.userInfo!, avatar: response.avatar_url }
    userStore.setUserInfo(updatedUser)

    ElMessage.success('头像上传成功')
  } catch (error: any) {
    ElMessage.error(error.response?.data?.detail || '头像上传失败')
  } finally {
    saving.value = false
  }
}

const handleChangePassword = async () => {
  if (!passwordFormRef.value) return

  try {
    await passwordFormRef.value.validate()
    changingPassword.value = true

    const data: PasswordChange = {
      old_password: passwordForm.old_password,
      new_password: passwordForm.new_password
    }

    await changePassword(data)
    ElMessage.success('密码修改成功')

    resetPasswordForm()
  } catch (error: any) {
    if (error.response?.status === 400) {
      ElMessage.error(error.response.data.detail || '原密码错误')
    } else {
      ElMessage.error('密码修改失败，请稍后重试')
    }
  } finally {
    changingPassword.value = false
  }
}

const resetPasswordForm = () => {
  passwordForm.old_password = ''
  passwordForm.new_password = ''
  passwordForm.confirm_password = ''
  passwordFormRef.value?.clearValidate()
}

const handleDeleteAccount = async () => {
  try {
    await deleteUser()
    ElMessage.success('账号已注销')

    setTimeout(() => {
      userStore.logout()
      window.location.href = '/login'
    }, 1500)
  } catch (error: any) {
    ElMessage.error('注销失败，请稍后重试')
  }
}

const formatDate = (dateStr?: string) => {
  if (!dateStr) return '未知'
  return new Date(dateStr).toLocaleString('zh-CN')
}

const triggerUpload = () => {
  const uploadEl = uploadRef.value?.$el?.querySelector('input[type="file"]')
  if (uploadEl) {
    uploadEl.click()
  }
}

onMounted(() => {
  loadUserInfo()
})
</script>

<style scoped>
.profile-container {
  padding: 24px;
  max-width: 1200px;
  margin: 0 auto;
  min-height: calc(100vh - 60px);
}

.profile-card {
  border-radius: 12px;
  overflow: hidden;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.header-title {
  font-size: 20px;
  font-weight: 600;
  color: #1f2937;
}

.profile-content {
  display: flex;
  gap: 32px;
  min-height: 600px;
}

.profile-sidebar {
  width: 280px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px 16px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  color: white;
}

.avatar-wrapper {
  position: relative;
  margin-bottom: 20px;
}

.user-avatar {
  border: 4px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
  cursor: pointer;
}

.user-avatar:hover {
  transform: scale(1.05);
  border-color: rgba(255, 255, 255, 0.6);
}

.avatar-uploader {
  position: absolute;
  bottom: 0;
  right: 0;
}

.upload-btn {
  background: white;
  border: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
  color: #667eea;
}

.upload-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.user-info-brief {
  text-align: center;
  width: 100%;
}

.username {
  margin: 0 0 8px 0;
  font-size: 20px;
  font-weight: 600;
  color: white;
}

.user-email {
  margin: 0 0 12px 0;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.9);
  word-break: break-all;
}

.bio-tag {
  background: rgba(255, 255, 255, 0.2) !important;
  border: none !important;
  color: white !important;
  max-width: 100%;
}

.profile-sidebar :deep(.el-divider) {
  border-color: rgba(255, 255, 255, 0.2);
  margin: 20px 0;
}

.profile-menu {
  width: 100%;
  background: transparent;
  border: none;
}

.profile-menu :deep(.el-menu-item) {
  color: rgba(255, 255, 255, 0.95);
  border-radius: 8px;
  margin-bottom: 8px;
  transition: all 0.3s ease;
  font-weight: 500;
}

.profile-menu :deep(.el-menu-item:hover) {
  background: rgba(255, 255, 255, 0.15);
  color: white;
}

.profile-menu :deep(.el-menu-item.is-active) {
  background: rgba(255, 255, 255, 0.25);
  color: white;
}

.profile-main {
  flex: 1;
  min-width: 0;
}

.content-panel {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 2px solid #f3f4f6;
}

.panel-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
}

.edit-actions {
  display: flex;
  gap: 12px;
}

.info-display {
  animation: fadeIn 0.3s ease;
}

.info-descriptions {
  background: white;
}

.info-value {
  font-size: 14px;
  color: #374151;
  line-height: 1.6;
}

.bio-text {
  white-space: pre-wrap;
  word-break: break-word;
}

.edit-form {
  max-width: 600px;
  animation: fadeIn 0.3s ease;
}

.password-form {
  max-width: 500px;
}

.security-descriptions {
  margin-bottom: 24px;
}

.desc-icon {
  margin-right: 8px;
  color: #6b7280;
  vertical-align: middle;
}

.danger-alert {
  border-radius: 8px;
}

.danger-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.danger-content p {
  margin: 0;
  flex: 1;
  color: #991b1b;
  font-size: 14px;
}

@media (max-width: 768px) {
  .profile-content {
    flex-direction: column;
  }

  .profile-sidebar {
    width: 100%;
  }

  .danger-content {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>
