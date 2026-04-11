<template>
  <div class="login-container">
    <div class="login-box">
      <h2 class="title">{{ isRegister ? 'Registration' : 'Login' }}</h2>
      
      <form @submit.prevent="handleSubmit">
        <div class="form-item">
          <label>User Name</label>
          <input 
            v-model="form.username" 
            type="text" 
            placeholder="Please enter your username"
            required
          />
        </div>
        
        <div class="form-item">
          <label>Password</label>
          <input 
            v-model="form.password" 
            type="password" 
            placeholder="Please enter the password"
            required
          />
        </div>
        
        <div class="form-item" v-if="isRegister">
          <label>Confirm Password</label>
          <input 
            v-model="form.confirmPassword" 
            type="password" 
            placeholder="Please confirm the password"
            required
          />
        </div>
        
        <div class="error-msg" v-if="errorMsg">{{ errorMsg }}</div>
        
        <button type="submit" class="submit-btn">
          {{ isRegister ? 'Register' : 'Login' }}
        </button>
      </form>
      
      <div class="switch-mode">
        <span @click="toggleMode">
          {{ isRegister ? 'Have an account? Login' : 'No account? Go to register' }}
        </span>
      </div>
      
      <div class="guest-access">
        <button @click="guestLogin" class="guest-btn">
          Tourist visit
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { useRouter } from 'vue-router'
import { ref, reactive } from 'vue'
import { indexedDBHelper, type UserInfo } from '../utils/indexDB'

export default {
  name: 'Login',
  setup() {
    const router = useRouter()
    const isRegister = ref(false)
    const errorMsg = ref('')
    const isLoading = ref(false)
    
    const form = reactive({
      username: '',
      password: '',
      confirmPassword: ''
    })

    const toggleMode = () => {
      isRegister.value = !isRegister.value
      errorMsg.value = ''
      form.password = ''
      form.confirmPassword = ''
    }

    const handleSubmit = async () => {
      errorMsg.value = ''
      isLoading.value = true
      
      try {
        // 注册逻辑
        if (isRegister.value) {
          // 密码一致性验证
          if (form.password !== form.confirmPassword) {
            errorMsg.value = 'The two password entries do not match'
            return
          }
          // 密码长度验证
          if (form.password.length < 6) {
            errorMsg.value = 'The password length should be at least 6 characters'
            return
          }
          
          // 检查用户名是否已存在
          const exists = await indexedDBHelper.usernameExists(form.username)
          if (exists) {
            errorMsg.value = 'The username already exists'
            return
          }
          
          // 创建用户对象
          const userInfo: UserInfo = {
            username: form.username,
            password: form.password,
            createTime: Date.now()
          }
          
          // 调用注册方法
          await indexedDBHelper.registerUser(userInfo)
          alert('Registration successful, please log in')
          toggleMode()
        } 
        // 登录逻辑
        else {
          // 调用验证方法
          const user = await indexedDBHelper.validateUser(form.username, form.password)
          
          // 保存当前用户信息到 localStorage
          localStorage.setItem('currentUser', JSON.stringify(user))
          
          // 跳转到主页面
          router.push('/TravelApp')
        }
      } catch (err: any) {
        errorMsg.value = err.message || 'Operation failed, please try again'
      } finally {
        isLoading.value = false
      }
    }

    const guestLogin = () => {
      const guestUser: UserInfo = {
        username: 'guest',
        password: '',
        createTime: Date.now(),
        isGuest: true
      }
      localStorage.setItem('currentUser', JSON.stringify(guestUser))
      router.push('/TravelApp')
    }

    return {
      isRegister,
      form,
      errorMsg,
      isLoading,
      toggleMode,
      handleSubmit,
      guestLogin
    }
  }
}
</script>

<style scoped>
.login-container {
  /* 使用风景图片作为背景 */
  background: url('https://img0.baidu.com/it/u=2175486724,1309616907&fm=253&fmt=auto&app=138&f=JPEG?w=713&h=475') no-repeat center center;
  background-size: cover;
  background-attachment: fixed; 

  min-height: 100vh;
  min-height: 100dvh;
  width: 100%;
  overflow-x: hidden; 
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f2f5; 
  /* 确保容器本身没有奇怪的 padding 或 margin 影响宽度 */
  padding: 0;
  margin: 0;
}

/* 背景遮罩层，让登录框更突出 */
.login-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(228, 229, 232, 0.7) 0%, rgba(139, 138, 139, 0.7) 100%);
  z-index: 0;
}

.login-box {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 50px 60px;
  border-radius: 15px;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.2);
  width: 90%;
  max-width: 550px;
  position: relative;
  z-index: 1;
  animation: fadeInUp 0.6s ease;
  margin: 20px;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.title {
  text-align: center;
  color: #4a6cf7;
  margin-bottom: 30px;
  font-size: 28px;
  font-weight: 600;
  position: relative;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 3px;
  background: linear-gradient(90deg, #4a6cf7, #6a11cb);
  border-radius: 3px;
}

.form-item {
  margin-bottom: 20px;
}

.form-item label {
  display: block;
  margin-bottom: 8px;
  color: #333;
  font-weight: 500;
  font-size: 15px;
}

.form-item input {
  width: 100%;
  padding: 14px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  font-size: 16px;
  box-sizing: border-box;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.9);
}

.form-item input:focus {
  outline: none;
  border-color: #4a6cf7;
  box-shadow: 0 0 0 3px rgba(74, 108, 247, 0.15);
}

.form-item input::placeholder {
  color: #999;
}

.error-msg {
  color: #ff6b6b;
  font-size: 14px;
  margin-bottom: 15px;
  text-align: center;
  padding: 10px;
  background: rgba(255, 107, 107, 0.1);
  border-radius: 5px;
  animation: shake 0.5s ease;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #7eb6ff 0%, #4a90e2 100%);
  color: white;
  border: none;
  border-radius: 30px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(74, 144, 226, 0.4);
  -webkit-tap-highlight-color: transparent;
}

.submit-btn:hover {
  background: linear-gradient(135deg, #92c4ff 0%, #5a9fef 100%);
  transform: translateY(-3px);
  box-shadow: 0 7px 20px rgba(74, 144, 226, 0.5);
}

.submit-btn:active {
  transform: translateY(-1px);
}

.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.switch-mode {
  text-align: center;
  margin-top: 20px;
  color: #4a6cf7;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  padding: 10px 0;
}

.switch-mode span:hover {
  text-decoration: underline;
  color: #6a11cb;
}

.guest-access {
  margin-top: 25px;
  text-align: center;
  padding-top: 20px;
  border-top: 1px solid #e0e0e0;
}

.guest-btn {
  padding: 12px 35px;
  background: transparent;
  color: #7d7e80;
  border: 2px solid #e3e3e5ea;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 15px;
  font-weight: 500;
  width: 100%;
  max-width: 200px; 
}

.guest-btn:hover {
  background: #4a6cf7;
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(74, 108, 247, 0.3);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .login-box {
    padding: 40px 30px;
    max-width: 85%;
  }
  
  .title {
    font-size: 24px;
  }
  
  .form-item input {
    padding: 12px;
    font-size: 16px;
  }
  
  .submit-btn {
    padding: 12px;
    font-size: 15px;
  }
}

@media (max-width: 480px) {
  .login-container {
    min-width: none;
    background-attachment: scroll; /* 移动端性能优化：滚动时不固定背景 */
  }

  .login-box {
    padding: 30px 20px;
    width: 90%;
    margin: 15px;
    border-radius: 12px;
  }
  
  .title {
    font-size: 22px;
    margin-bottom: 20px;
  }
  
  .form-item {
    margin-bottom: 15px;
  }

  .form-item label {
    font-size: 14px;
  }
  
  .guest-btn {
    padding: 10px 25px;
    font-size: 14px;
    max-width: 100%; /* 手机端游客按钮全宽 */
  }

  .switch-mode {
    font-size: 13px;
  }
}

/* 超小屏幕适配 */
@media (max-width: 320px) {
  .login-box {
    padding: 20px 15px;
  }
  
  .title {
    font-size: 20px;
  }
}
</style>