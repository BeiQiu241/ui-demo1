<template>
  <div class="login-container">
    <!-- 左侧：容器 -->
    <div class="character-side">
      <!-- 只有这个 wrapper 是闭眼触发区 -->
      <div 
        class="character-wrapper"
        @mouseenter="isHoveringCharacter = true" 
        @mouseleave="isHoveringCharacter = false"
        :style="bodyStyle"
      >
        <!-- 极简 SVG 小人 -->
        <svg viewBox="0 0 200 300" class="character-svg">
          <defs>
            <clipPath id="eyeClipLarge">
              <circle cx="0" cy="0" r="22" />
            </clipPath>
          </defs>
          
          <g class="character-main-group">
            <!-- 身体/脑袋 (变得更宽、更饱满，线条加粗到 6) -->
            <path d="M30,270 Q100,20 170,270" fill="none" stroke="#333" stroke-width="6" stroke-linecap="round"/>
            
            <!-- 左眼 (位置更靠外，高度微调) -->
            <g transform="translate(65, 135)" class="eye-group">
              <!-- 睁眼/半睁状态内容 -->
              <g :class="['eye-open-container', { 'hidden': isLeftEyeClosed }]">
                <circle cx="0" cy="0" r="22" fill="white" stroke="#333" stroke-width="4" />
                <g clip-path="url(#eyeClipLarge)">
                  <circle :style="leftEyeStyle" cx="0" cy="0" r="10" fill="#333" />
                  <rect 
                    x="-30" 
                    :y="isLeftEyeHalf ? -40 : -80" 
                    width="60" 
                    height="40" 
                    fill="#333" 
                    class="eyelid-rect"
                  />
                </g>
              </g>
              <!-- 全闭状态内容 (横线加长加粗) -->
              <path 
                class="eye-closed-line"
                :class="{ 'visible': isLeftEyeClosed }"
                d="M -18,0 L 18,0" 
                stroke="#333" 
                stroke-width="5" 
                stroke-linecap="round" 
              />
            </g>

            <!-- 右眼 -->
            <g transform="translate(135, 135)" class="eye-group">
              <g :class="['eye-open-container', { 'hidden': isRightEyeClosed }]">
                <circle cx="0" cy="0" r="22" fill="white" stroke="#333" stroke-width="4" />
                <g clip-path="url(#eyeClipLarge)">
                  <circle :style="rightEyeStyle" cx="0" cy="0" r="10" fill="#333" />
                  <rect 
                    x="-30" 
                    :y="isRightEyeHalf ? -40 : -80" 
                    width="60" 
                    height="40" 
                    fill="#333" 
                    class="eyelid-rect"
                  />
                </g>
              </g>
              <path 
                class="eye-closed-line"
                :class="{ 'visible': isRightEyeClosed }"
                d="M -18,0 L 18,0" 
                stroke="#333" 
                stroke-width="5" 
                stroke-linecap="round" 
              />
            </g>

            <!-- 嘴巴 (位置和宽度适配大身体) -->
            <path 
              :d="mouthPath" 
              fill="none" 
              stroke="#333" 
              stroke-width="4" 
              stroke-linecap="round"
              class="mouth-path"
            />
          </g>
        </svg>
      </div>
    </div>

    <!-- 右侧：登录表单 -->
    <div class="form-side">
      <div class="login-box" @mouseenter="isLeftEyeClosed = false; isRightEyeClosed = false">
        <h1>Welcome Back</h1>
        <p class="subtitle">Please sign in to continue</p>
        
        <form @submit.prevent="handleLogin">
          <div class="input-group">
            <label>Username</label>
            <input 
              type="text" 
              placeholder="Enter your username" 
              v-model="form.username" 
              required 
              @mouseenter="isHoveringInputs = true"
              @mouseleave="isHoveringInputs = false"
            />
          </div>
          
          <div class="input-group">
            <label>Password</label>
            <input 
              type="password" 
              placeholder="••••••••" 
              v-model="form.password" 
              required 
              @mouseenter="isHoveringInputs = true"
              @mouseleave="isHoveringInputs = false"
            />
          </div>

          <div class="options">
            <label class="remember">
              <input type="checkbox" /> Remember me
            </label>
            <a href="#" class="forgot">Forgot Password?</a>
          </div>

          <button type="submit" class="login-btn">Sign In</button>
        </form>

        <div class="footer">
          Don't have an account? <a href="#">Create Account</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, computed } from 'vue'

const form = reactive({
  username: '',
  password: ''
})

// 基础状态定义
const isLeftEyeClosed = ref(false)
const isRightEyeClosed = ref(false)
const isLeftEyeHalf = ref(false)
const isRightEyeHalf = ref(false)
const isHoveringCharacter = ref(false)
const isHoveringInputs = ref(false)
const moveDirection = ref('left')
let lastMouseX = 0

// 坐标状态
const leftEyePos = reactive({ x: 0, y: 0 })
const rightEyePos = reactive({ x: 0, y: 0 })
const mousePos = reactive({ x: 0, y: 0 })

// 根据眼睛状态计算嘴巴路径
const mouthPath = computed(() => {
  if (isLeftEyeClosed.value && isRightEyeClosed.value) {
    return 'M85,192 Q100,192 115,192' 
  }
  if (isLeftEyeHalf.value && isRightEyeHalf.value) {
    return 'M88,190 Q100,194 112,190' 
  }
  if (isLeftEyeClosed.value) {
    return 'M85,188 Q105,198 115,182'
  }
  if (isRightEyeClosed.value) {
    return 'M85,182 Q95,198 115,188'
  }
  return 'M82,185 Q100,205 118,185' 
})

// 计算瞳孔偏移
const calculateOffset = (eyeX, eyeY, isHalf) => {
  const dx = mousePos.x - eyeX
  const dy = mousePos.y - eyeY
  const angle = Math.atan2(dy, dx)
  
  const maxDist = 11
  const distance = Math.min(Math.hypot(dx, dy) / 12, maxDist) 
  
  let offsetX = Math.cos(angle) * distance
  let offsetY = Math.sin(angle) * distance
  
  if (isHalf) {
    offsetY = Math.max(-2, Math.min(offsetY, 8)) 
  }
  
  return {
    transform: `translate(${offsetX}px, ${offsetY}px)`,
    transition: 'transform 0.1s ease-out' 
  }
}

const leftEyeStyle = computed(() => calculateOffset(leftEyePos.x, leftEyePos.y, isLeftEyeHalf.value))
const rightEyeStyle = computed(() => calculateOffset(rightEyePos.x, rightEyePos.y, isRightEyeHalf.value))

// 身体重心随动
const bodyStyle = computed(() => {
  const width = window.innerWidth
  const centerX = width / 2
  const tiltAngle = ((mousePos.x - centerX) / centerX) * 5
  
  return {
    transform: `rotate(${tiltAngle}deg)`,
    transition: 'transform 0.4s cubic-bezier(0.2, 0, 0.2, 1)'
  }
})

// 眼睛位置更新
const updateEyePositions = () => {
  const circles = document.querySelectorAll('.eye-group circle[fill="white"]')
  const leftEyeEl = circles[0]
  const rightEyeEl = circles[1]
  
  if (leftEyeEl && rightEyeEl) {
    const leftRect = leftEyeEl.getBoundingClientRect()
    const rightRect = rightEyeEl.getBoundingClientRect()
    
    leftEyePos.x = leftRect.left + leftRect.width / 2
    leftEyePos.y = leftRect.top + leftRect.height / 2
    
    rightEyePos.x = rightRect.left + rightRect.width / 2
    rightEyePos.y = rightRect.top + rightRect.height / 2
  }
}

const handleMouseMove = (e) => {
  const currentX = e.clientX
  const deltaX = currentX - lastMouseX
  mousePos.x = currentX
  mousePos.y = e.clientY
  
  if (Math.abs(deltaX) > 5) {
    moveDirection.value = deltaX > 0 ? 'right' : 'left'
  }

  const width = window.innerWidth
  const RESET_ZONE = width * 0.05
  const CURIOUS_ZONE = width * 0.25
  const SHY_ZONE_END = width * 0.45
  const TRANSITION_LINE = width * 0.55

  // 优先级逻辑
  if (isHoveringCharacter.value) {
    isLeftEyeClosed.value = true; isRightEyeClosed.value = true
    isLeftEyeHalf.value = false; isRightEyeHalf.value = false
  } else if (isHoveringInputs.value) {
    isLeftEyeClosed.value = true; isRightEyeClosed.value = false
    isLeftEyeHalf.value = false; isRightEyeHalf.value = true
  } else if (currentX < RESET_ZONE) {
    isLeftEyeClosed.value = false; isRightEyeClosed.value = false
    isLeftEyeHalf.value = false; isRightEyeHalf.value = false
  } else if (currentX < CURIOUS_ZONE) {
    if (moveDirection.value === 'right') {
      isLeftEyeClosed.value = true; isRightEyeClosed.value = false
      isLeftEyeHalf.value = false; isRightEyeHalf.value = true
    } else {
      isLeftEyeClosed.value = false; isRightEyeClosed.value = false
      isLeftEyeHalf.value = false; isRightEyeHalf.value = false
    }
  } else if (currentX < SHY_ZONE_END) {
    isLeftEyeClosed.value = true; isRightEyeClosed.value = true
    isLeftEyeHalf.value = false; isRightEyeHalf.value = false
  } else if (currentX < TRANSITION_LINE) {
    if (moveDirection.value === 'right') { 
      isLeftEyeClosed.value = true; isRightEyeClosed.value = false
      isLeftEyeHalf.value = false; isRightEyeHalf.value = true
    } else { 
      isLeftEyeClosed.value = false; isRightEyeClosed.value = true
      isLeftEyeHalf.value = true; isRightEyeHalf.value = false
    }
  } else {
    isLeftEyeClosed.value = false; isRightEyeClosed.value = false
    isLeftEyeHalf.value = false; isRightEyeHalf.value = false
  }
  
  lastMouseX = currentX
}

onMounted(() => {
  updateEyePositions()
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('resize', updateEyePositions)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('resize', updateEyePositions)
})

const handleLogin = () => {
  alert('Login successful!')
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');

.login-container {
  display: flex;
  height: 100vh;
  width: 100vw;
  background-color: #f8f9fa;
  font-family: 'Inter', sans-serif;
  overflow: hidden;
}

.character-side {
  flex: 1.2;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #ffffff;
  pointer-events: none;
}

.character-wrapper {
  width: fit-content;
  max-width: 600px;
  pointer-events: auto;
  cursor: pointer;
  animation: breathing 4s ease-in-out infinite;
}

@keyframes breathing {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.02); }
}

.character-svg {
  width: 100%;
  filter: drop-shadow(0 10px 15px rgba(0,0,0,0.05));
}

.mouth-path {
  transition: d 0.3s ease;
}

.eye-open-container {
  transition: opacity 0.3s ease;
  opacity: 1;
}

.eye-open-container.hidden {
  opacity: 0;
  pointer-events: none;
}

.eye-closed-line {
  transition: opacity 0.3s ease;
  opacity: 0;
  pointer-events: none;
}

.eye-closed-line.visible {
  opacity: 1;
}

.eyelid-rect {
  transition: y 0.3s ease-in-out;
}

.form-side {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

.login-box {
  width: 100%;
  max-width: 420px;
  background: white;
  padding: 40px;
  border-radius: 24px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.04);
}

h1 {
  font-size: 28px;
  font-weight: 600;
  color: #1a1a1a;
  margin-bottom: 8px;
}

.subtitle {
  color: #666;
  margin-bottom: 32px;
  font-size: 15px;
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 8px;
}

.input-group input {
  width: 100%;
  padding: 12px 16px;
  border-radius: 12px;
  border: 1px solid #e1e1e1;
  font-size: 15px;
  transition: all 0.2s;
  box-sizing: border-box;
}

.input-group input:focus {
  outline: none;
  border-color: #333;
  box-shadow: 0 0 0 4px rgba(0,0,0,0.03);
}

.options {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 32px;
  font-size: 14px;
}

.remember {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #666;
  cursor: pointer;
}

.forgot {
  color: #333;
  text-decoration: none;
  font-weight: 600;
}

.login-btn {
  width: 100%;
  padding: 14px;
  background-color: #1a1a1a;
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.login-btn:hover {
  background-color: #333;
}

.footer {
  text-align: center;
  margin-top: 32px;
  font-size: 14px;
  color: #666;
}

.footer a {
  color: #1a1a1a;
  text-decoration: none;
  font-weight: 600;
}

@media (max-width: 900px) {
  .login-container {
    flex-direction: column;
  }
  .character-side {
    flex: 0.5;
    padding-top: 20px;
  }
}
</style>
