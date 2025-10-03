<template>
  <div class="tech-website">
    <!-- 流星效果 -->
    <div class="meteor" v-for="n in 5" :key="`meteor-${n}`"></div>
    
    <!-- 粒子背景容器 -->
    <div id="particles-js" ref="particlesContainer"></div>
    
    <!-- 浮动科技元素 -->
    <div class="floating-elements">
      <div class="tech-element" v-for="n in 3" :key="`tech-${n}`"></div>
    </div>
    
    <!-- 流体云雾Canvas -->
    <canvas id="fluidCanvas" ref="fluidCanvas"></canvas>
    
    <!-- 鼠标跟随效果 -->
    <div class="cursor-main" ref="cursorMain"></div>
    
    <!-- 导航栏 -->
    <div class="container">
      <nav>
        <div class="logo">科技公司</div>
        <ul class="nav-links">
          <li><a href="#home">首页</a></li>
          <li><a href="#features">产品特性</a></li>
          <li><a href="#about">关于我们</a></li>
          <li><a href="#contact">联系我们</a></li>
        </ul>
        <button class="cta-button" @click="handleButtonClick">免费试用</button>
      </nav>
    </div>
    
    <!-- 英雄区域 -->
    <section class="hero" id="home">
      <div class="container">
        <div class="hero-content" ref="heroContent">
          <h1 data-text="创新科技，驱动未来">创新科技，驱动未来</h1>
          <p>ai真好用。点下下面按钮试试。你猜我用了几个模型？<span class="typed-cursor">|</span></p>
          <div class="hero-buttons">
            <button class="cta-button" @click="handleButtonClick">开始体验</button>
            <button class="outline-button" @click="handleButtonClick">了解更多</button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

// Refs
const particlesContainer = ref(null);
const fluidCanvas = ref(null);
const cursorMain = ref(null);
const heroContent = ref(null);

// 状态变量
let mouseX = 0, mouseY = 0;
let cursorMainX = 0, cursorMainY = 0;
let lastMouseX = 0, lastMouseY = 0;
let isDragging = false;
let animationFrameId = null;
let ctx = null;
const fluidParticles = [];
const maxParticles = 100; // 降低粒子数量
let lastParticleTime = 0;
let lastDOMParticleTime = 0;
let frameCount = 0;
let lastFpsTime = Date.now();
let fps = 60;

// 流体粒子类
class FluidParticle {
  constructor(x, y, isDrag = false) {
    this.x = x;
    this.y = y;
    this.vx = (Math.random() - 0.5) * (isDrag ? 5 : 3);
    this.vy = (Math.random() - 0.5) * (isDrag ? 5 : 3);
    this.size = Math.random() * (isDrag ? 80 : 60) + (isDrag ? 50 : 40);
    this.life = 1;
    this.decay = Math.random() * 0.01 + (isDrag ? 0.003 : 0.005);
    this.color = ['rgba(0, 255, 255,', 'rgba(0, 198, 255,', 'rgba(255, 0, 255,', 'rgba(138, 43, 226,'][Math.floor(Math.random() * 4)];
    this.isDrag = isDrag;
  }
  
  update() {
    this.x += this.vx;
    this.y += this.vy;
    this.vx *= 0.98;
    this.vy *= 0.98;
    this.life -= this.decay;
    this.size += this.isDrag ? 1 : 0.5;
  }
  
  draw() {
    const gradient = ctx.createRadialGradient(this.x, this.y, 0, this.x, this.y, this.size);
    gradient.addColorStop(0, this.color + (this.life * (this.isDrag ? 0.8 : 0.6)) + ')');
    gradient.addColorStop(0.5, this.color + (this.life * (this.isDrag ? 0.5 : 0.3)) + ')');
    gradient.addColorStop(1, this.color + '0)');
    
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fill();
  }
}

// 调整Canvas大小
const resizeCanvas = () => {
  if (fluidCanvas.value) {
    fluidCanvas.value.width = window.innerWidth;
    fluidCanvas.value.height = window.innerHeight;
  }
};

// 创建云雾拖尾
const createCloudTrail = (x, y, speed, isDrag = false) => {
  const cloud = document.createElement('div');
  cloud.className = 'cloud-trail';
  const size = Math.random() * (isDrag ? 150 : 100) + speed * 2;
  cloud.style.width = size + 'px';
  cloud.style.height = size + 'px';
  cloud.style.left = x - size / 2 + 'px';
  cloud.style.top = y - size / 2 + 'px';
  
  const colors = [
    'radial-gradient(circle, rgba(0, 255, 255, 0.6), transparent)',
    'radial-gradient(circle, rgba(0, 198, 255, 0.6), transparent)',
    'radial-gradient(circle, rgba(255, 0, 255, 0.6), transparent)',
    'radial-gradient(circle, rgba(138, 43, 226, 0.6), transparent)'
  ];
  cloud.style.background = colors[Math.floor(Math.random() * colors.length)];
  
  if (isDrag) {
    cloud.style.animationDuration = '3s';
  }
  
  document.body.appendChild(cloud);
  setTimeout(() => cloud.remove(), isDrag ? 3000 : 2000);
};

// 创建流体粒子DOM
const createFluidParticleDOM = (x, y, isDrag = false) => {
  const particle = document.createElement('div');
  particle.className = 'fluid-particle';
  const size = Math.random() * (isDrag ? 30 : 20) + (isDrag ? 15 : 10);
  particle.style.width = size + 'px';
  particle.style.height = size + 'px';
  particle.style.left = x - size / 2 + 'px';
  particle.style.top = y - size / 2 + 'px';
  
  const colors = [
    'radial-gradient(circle, rgba(0, 255, 255, 1), transparent)',
    'radial-gradient(circle, rgba(255, 0, 255, 1), transparent)',
    'radial-gradient(circle, rgba(0, 198, 255, 1), transparent)'
  ];
  particle.style.background = colors[Math.floor(Math.random() * colors.length)];
  particle.style.boxShadow = `0 0 ${isDrag ? 50 : 30}px ${['#00ffff', '#ff00ff', '#00c6ff'][Math.floor(Math.random() * 3)]}`;
  
  const angle = Math.random() * Math.PI * 2;
  const distance = Math.random() * (isDrag ? 150 : 100) + (isDrag ? 80 : 50);
  const tx = Math.cos(angle) * distance;
  const ty = Math.sin(angle) * distance;
  
  particle.style.setProperty('--tx', tx + 'px');
  particle.style.setProperty('--ty', ty + 'px');
  particle.style.animationDuration = (Math.random() * 1 + (isDrag ? 1.5 : 1)) + 's';
  
  document.body.appendChild(particle);
  setTimeout(() => particle.remove(), isDrag ? 2500 : 2000);
};

// 创建拖动轨迹特效
const createDragTrail = (x, y) => {
  const trail = document.createElement('div');
  trail.className = 'drag-trail';
  const size = Math.random() * 40 + 30;
  trail.style.width = size + 'px';
  trail.style.height = size + 'px';
  trail.style.left = x - size / 2 + 'px';
  trail.style.top = y - size / 2 + 'px';
  
  const colors = ['#00ffff', '#ff00ff', '#00c6ff', '#8a2be2'];
  trail.style.background = `radial-gradient(circle, ${colors[Math.floor(Math.random() * colors.length)]}, transparent)`;
  trail.style.boxShadow = `0 0 40px ${colors[Math.floor(Math.random() * colors.length)]}`;
  
  document.body.appendChild(trail);
  setTimeout(() => trail.remove(), 1000);
};

// 点击爆炸特效
const createExplosion = (x, y) => {
  const colors = ['#00ffff', '#ff00ff', '#0072ff', '#00ff00', '#ffff00'];
  const particleCount = 30;
  
  for (let i = 0; i < particleCount; i++) {
    const particle = document.createElement('div');
    particle.className = 'explosion-particle';
    particle.style.left = x + 'px';
    particle.style.top = y + 'px';
    particle.style.background = colors[Math.floor(Math.random() * colors.length)];
    particle.style.boxShadow = `0 0 10px ${colors[Math.floor(Math.random() * colors.length)]}`;
    
    const angle = (Math.PI * 2 * i) / particleCount;
    const velocity = 50 + Math.random() * 100;
    const tx = Math.cos(angle) * velocity;
    const ty = Math.sin(angle) * velocity;
    
    particle.style.setProperty('--tx', `translate(${tx}px, ${ty}px)`);
    
    document.body.appendChild(particle);
    setTimeout(() => particle.remove(), 1000);
  }
};

// 鼠标移动处理
const handleMouseMove = (e) => {
  mouseX = e.clientX;
  mouseY = e.clientY;
  
  const dx = mouseX - lastMouseX;
  const dy = mouseY - lastMouseY;
  const speed = Math.sqrt(dx * dx + dy * dy);
  
  // 根据速度和拖动状态生成粒子
  const particleThreshold = isDragging ? 1 : 2;
  const particleMultiplier = isDragging ? 5 : 3;
  
  if (speed > particleThreshold) {
    const particleCount = Math.min(particleMultiplier, Math.floor(speed / (isDragging ? 5 : 10)));
    
    for (let i = 0; i < particleCount; i++) {
      if (fluidParticles.length < maxParticles) {
        fluidParticles.push(new FluidParticle(mouseX, mouseY, isDragging));
      }
    }
    
    // 拖动时产生更多云雾
    if (Math.random() > (isDragging ? 0.3 : 0.6)) {
      createCloudTrail(mouseX, mouseY, speed, isDragging);
    }
    
    // 拖动时产生更多流体粒子
    if (Math.random() > (isDragging ? 0.5 : 0.7)) {
      createFluidParticleDOM(mouseX, mouseY, isDragging);
    }
    
    // 拖动特效
    if (isDragging && Math.random() > 0.5) {
      createDragTrail(mouseX, mouseY);
    }
  }
  
  lastMouseX = mouseX;
  lastMouseY = mouseY;
  
  // 3D倾斜效果
  if (heroContent.value && window.innerWidth > 768) {
    const xAxis = (window.innerWidth / 2 - e.pageX) / 50;
    const yAxis = (window.innerHeight / 2 - e.pageY) / 50;
    heroContent.value.style.transform = `rotateY(${xAxis}deg) rotateX(${yAxis}deg)`;
  }
};

// 鼠标按下
const handleMouseDown = (e) => {
  isDragging = true;
  if (cursorMain.value) {
    cursorMain.value.style.transform = 'scale(1.5)';
    cursorMain.value.style.boxShadow = `
      0 0 50px #00ffff,
      0 0 100px #00ffff,
      0 0 150px #00ffff
    `;
  }
};

// 鼠标松开
const handleMouseUp = (e) => {
  if (isDragging) {
    createExplosion(e.clientX, e.clientY);
  }
  isDragging = false;
  if (cursorMain.value) {
    cursorMain.value.style.transform = 'scale(1)';
    cursorMain.value.style.boxShadow = `
      0 0 30px #00ffff,
      0 0 60px #00ffff,
      0 0 90px #00ffff
    `;
  }
};

// 点击事件
const handleClick = (e) => {
  createExplosion(e.clientX, e.clientY);
};

// 平滑跟随动画
const animateCursor = () => {
  cursorMainX += (mouseX - cursorMainX) * 0.15;
  cursorMainY += (mouseY - cursorMainY) * 0.15;
  
  if (cursorMain.value) {
    cursorMain.value.style.left = cursorMainX - 15 + 'px';
    cursorMain.value.style.top = cursorMainY - 15 + 'px';
  }
  
  animationFrameId = requestAnimationFrame(animateCursor);
};

// 渲染流体
const renderFluid = () => {
  if (!ctx) return;
  
  ctx.fillStyle = 'rgba(0, 0, 0, 0.05)';
  ctx.fillRect(0, 0, fluidCanvas.value.width, fluidCanvas.value.height);
  
  for (let i = fluidParticles.length - 1; i >= 0; i--) {
    const particle = fluidParticles[i];
    particle.update();
    particle.draw();
    
    if (particle.life <= 0) {
      fluidParticles.splice(i, 1);
    }
  }
  
  fluidAnimationFrameId = requestAnimationFrame(renderFluid);
};

// 按钮点击处理
const handleButtonClick = (e) => {
  const button = e.currentTarget;
  const ripple = document.createElement('span');
  const rect = button.getBoundingClientRect();
  const size = Math.max(rect.width, rect.height);
  const x = e.clientX - rect.left - size / 2;
  const y = e.clientY - rect.top - size / 2;
  
  ripple.style.width = ripple.style.height = size + 'px';
  ripple.style.left = x + 'px';
  ripple.style.top = y + 'px';
  ripple.style.position = 'absolute';
  ripple.style.borderRadius = '50%';
  ripple.style.background = 'rgba(255, 255, 255, 0.6)';
  ripple.style.transform = 'scale(0)';
  ripple.style.animation = 'ripple 0.6s ease-out';
  ripple.style.pointerEvents = 'none';
  
  button.style.position = 'relative';
  button.style.overflow = 'hidden';
  button.appendChild(ripple);
  
  setTimeout(() => ripple.remove(), 600);
  setTimeout(() => window.open('https://www.spacex.com', '_blank'), 200);
};

// 初始化粒子效果
const initParticles = () => {
  if (window.particlesJS && particlesContainer.value) {
    window.particlesJS('particles-js', {
      particles: {
        number: { value: 200, density: { enable: true, value_area: 800 } },
        color: { value: ["#00c6ff", "#0072ff", "#ff00ff", "#00ffff"] },
        shape: { type: ["circle", "triangle", "polygon"], polygon: { nb_sides: 5 } },
        opacity: {
          value: 0.7,
          random: true,
          anim: { enable: true, speed: 1, opacity_min: 0.1, sync: false }
        },
        size: {
          value: 4,
          random: true,
          anim: { enable: true, speed: 3, size_min: 0.1, sync: false }
        },
        line_linked: {
          enable: true,
          distance: 150,
          color: "#0072ff",
          opacity: 0.5,
          width: 2
        },
        move: {
          enable: true,
          speed: 5,
          direction: "none",
          random: true,
          straight: false,
          out_mode: "bounce",
          bounce: true,
          attract: { enable: true, rotateX: 600, rotateY: 1200 }
        }
      },
      interactivity: {
        detect_on: "canvas",
        events: {
          onhover: { enable: true, mode: ["grab", "bubble"] },
          onclick: { enable: true, mode: "repulse" },
          resize: true
        },
        modes: {
          grab: { distance: 250, line_linked: { opacity: 1 } },
          bubble: { distance: 250, size: 8, duration: 2, opacity: 0.9, speed: 3 },
          repulse: { distance: 250, duration: 0.4 }
        }
      },
      retina_detect: true
    });
  }
};

// 加载particles.js库
const loadParticlesJS = () => {
  return new Promise((resolve, reject) => {
    if (window.particlesJS) {
      resolve();
      return;
    }
    
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/particles.js/2.0.0/particles.min.js';
    script.onload = resolve;
    script.onerror = reject;
    document.head.appendChild(script);
  });
};

// 组件挂载
onMounted(async () => {
  // 初始化Canvas
  if (fluidCanvas.value) {
    ctx = fluidCanvas.value.getContext('2d');
    resizeCanvas();
  }
  
  // 添加事件监听
  window.addEventListener('resize', resizeCanvas);
  document.addEventListener('mousemove', handleMouseMove);
  document.addEventListener('mousedown', handleMouseDown);
  document.addEventListener('mouseup', handleMouseUp);
  document.addEventListener('click', handleClick);
  
  // 启动动画
  animateCursor();
  renderFluid();
  
  // 加载并初始化particles.js
  try {
    await loadParticlesJS();
    initParticles();
  } catch (error) {
    console.error('Failed to load particles.js:', error);
  }
});

// 组件卸载
onBeforeUnmount(() => {
  // 清理事件监听
  window.removeEventListener('resize', resizeCanvas);
  document.removeEventListener('mousemove', handleMouseMove);
  document.removeEventListener('mousedown', handleMouseDown);
  document.removeEventListener('mouseup', handleMouseUp);
  document.removeEventListener('click', handleClick);
  
  // 取消动画帧
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
  }
  if (fluidAnimationFrameId) {
    cancelAnimationFrame(fluidAnimationFrameId);
  }
  
  // 清理粒子
  fluidParticles.length = 0;
});
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', 'Microsoft YaHei', sans-serif;
}

.tech-website {
  background: linear-gradient(135deg, #000 0%, #0a0a2e 50%, #000 100%);
  color: #fff;
  overflow-x: hidden;
  line-height: 1.6;
  cursor: none;
  min-height: 100vh;
}

/* 背景动画 */
.tech-website::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 20% 50%, rgba(0, 198, 255, 0.15), transparent 50%),
              radial-gradient(circle at 80% 80%, rgba(0, 114, 255, 0.15), transparent 50%),
              radial-gradient(circle at 40% 20%, rgba(255, 0, 198, 0.1), transparent 50%);
  animation: backgroundPulse 15s ease-in-out infinite;
  z-index: -2;
}

@keyframes backgroundPulse {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 1; }
}

#particles-js {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: -1;
}

/* 流星效果 */
.meteor {
  position: fixed;
  top: -100px;
  width: 3px;
  height: 150px;
  background: linear-gradient(to bottom, transparent, rgba(0, 255, 255, 1), transparent);
  animation: meteorFall linear infinite;
  z-index: -1;
  filter: drop-shadow(0 0 10px rgba(0, 255, 255, 0.8));
}

.meteor:nth-child(1) { left: 10%; animation-duration: 8s; animation-delay: 0s; }
.meteor:nth-child(2) { left: 30%; animation-duration: 10s; animation-delay: 2s; }
.meteor:nth-child(3) { left: 50%; animation-duration: 12s; animation-delay: 4s; }
.meteor:nth-child(4) { left: 70%; animation-duration: 9s; animation-delay: 1s; }
.meteor:nth-child(5) { left: 90%; animation-duration: 11s; animation-delay: 3s; }

@keyframes meteorFall {
  to {
    transform: translateY(calc(100vh + 200px)) translateX(200px) rotate(45deg);
  }
}

/* 流体云雾效果 */
#fluidCanvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 9999;
  mix-blend-mode: screen;
}

.cursor-main {
  position: fixed;
  width: 30px;
  height: 30px;
  background: radial-gradient(circle, #00ffff, #00ffff 30%, transparent);
  border-radius: 50%;
  pointer-events: none;
  z-index: 10000;
  transition: transform 0.2s ease-out, box-shadow 0.2s ease-out;
  box-shadow: 0 0 30px #00ffff,
              0 0 60px #00ffff,
              0 0 90px #00ffff;
  mix-blend-mode: screen;
}

/* 流体粒子 */
:deep(.fluid-particle) {
  position: fixed;
  border-radius: 50%;
  pointer-events: none;
  z-index: 9998;
  mix-blend-mode: screen;
  animation: fluidFloat linear forwards;
}

@keyframes fluidFloat {
  0% {
    opacity: 1;
    transform: translate(0, 0) scale(1);
  }
  100% {
    opacity: 0;
    transform: translate(var(--tx), var(--ty)) scale(3);
  }
}

/* 云雾拖尾 */
:deep(.cloud-trail) {
  position: fixed;
  border-radius: 50%;
  pointer-events: none;
  z-index: 9997;
  mix-blend-mode: screen;
  animation: cloudExpand 2s ease-out forwards;
}

@keyframes cloudExpand {
  0% {
    opacity: 0.8;
    transform: scale(1);
    filter: blur(20px);
  }
  100% {
    opacity: 0;
    transform: scale(5);
    filter: blur(80px);
  }
}

/* 拖动轨迹 */
:deep(.drag-trail) {
  position: fixed;
  border-radius: 50%;
  pointer-events: none;
  z-index: 9996;
  mix-blend-mode: screen;
  animation: dragTrailFade 1s ease-out forwards;
  filter: blur(15px);
}

@keyframes dragTrailFade {
  0% {
    opacity: 1;
    transform: scale(1);
  }
  100% {
    opacity: 0;
    transform: scale(3);
  }
}

/* 爆炸粒子 */
:deep(.explosion-particle) {
  position: fixed;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  pointer-events: none;
  z-index: 10001;
  animation: explode 1s ease-out forwards;
}

@keyframes explode {
  0% {
    opacity: 1;
    transform: translate(0, 0) scale(1);
  }
  100% {
    opacity: 0;
    transform: var(--tx) scale(0);
  }
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* 导航栏 */
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 25px 0;
  position: relative;
  z-index: 100;
  backdrop-filter: blur(10px);
  animation: fadeInDown 1s ease-out;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.logo {
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(90deg, #00c6ff, #0072ff, #ff00ff, #00c6ff);
  background-size: 300% 100%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 1px;
  animation: gradientShift 3s ease infinite;
  cursor: pointer;
  transition: transform 0.3s;
  text-shadow: 0 0 30px rgba(0, 198, 255, 0.5);
}

.logo:hover {
  transform: scale(1.1) rotateZ(-5deg);
  filter: drop-shadow(0 0 20px rgba(0, 198, 255, 0.8));
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.nav-links {
  display: flex;
  list-style: none;
}

.nav-links li {
  margin-left: 35px;
  animation: fadeInUp 0.6s ease-out backwards;
}

.nav-links li:nth-child(1) { animation-delay: 0.1s; }
.nav-links li:nth-child(2) { animation-delay: 0.2s; }
.nav-links li:nth-child(3) { animation-delay: 0.3s; }
.nav-links li:nth-child(4) { animation-delay: 0.4s; }

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.nav-links a {
  color: #fff;
  text-decoration: none;
  font-weight: 500;
  font-size: 16px;
  transition: all 0.3s;
  position: relative;
  display: inline-block;
}

.nav-links a:hover {
  color: #00c6ff;
  text-shadow: 0 0 15px rgba(0, 198, 255, 0.8),
               0 0 30px rgba(0, 198, 255, 0.5);
  transform: translateY(-3px);
}

.nav-links a::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 50%;
  width: 0;
  height: 3px;
  background: linear-gradient(90deg, #00c6ff, #0072ff, #ff00ff);
  transition: all 0.3s;
  transform: translateX(-50%);
  box-shadow: 0 0 10px rgba(0, 198, 255, 0.8);
}

.nav-links a:hover::after {
  width: 100%;
}

/* 按钮样式 */
.cta-button {
  background: linear-gradient(45deg, #00c6ff, #0072ff, #ff00ff, #00c6ff);
  background-size: 400% 400%;
  color: white;
  border: none;
  padding: 14px 32px;
  border-radius: 30px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 15px;
  position: relative;
  overflow: hidden;
  animation: neonPulse 3s ease-in-out infinite, gradientShift 3s ease infinite;
  box-shadow: 0 0 10px rgba(0, 198, 255, 0.5),
              0 0 20px rgba(0, 198, 255, 0.3),
              0 0 30px rgba(0, 198, 255, 0.2),
              inset 0 0 10px rgba(255, 255, 255, 0.2);
}

@keyframes neonPulse {
  0%, 100% { 
    box-shadow: 0 0 10px rgba(0, 198, 255, 0.5),
               0 0 20px rgba(0, 198, 255, 0.3),
               0 0 30px rgba(0, 198, 255, 0.2),
               inset 0 0 10px rgba(255, 255, 255, 0.2);
  }
  50% { 
    box-shadow: 0 0 20px rgba(0, 198, 255, 0.8),
               0 0 40px rgba(0, 198, 255, 0.6),
               0 0 60px rgba(0, 198, 255, 0.4),
               inset 0 0 20px rgba(255, 255, 255, 0.3);
  }
}

.cta-button::after {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transform: rotate(45deg);
  animation: shine 3s infinite;
}

@keyframes shine {
  0% { transform: translateX(-100%) translateY(-100%) rotate(45deg); }
  100% { transform: translateX(100%) translateY(100%) rotate(45deg); }
}

.cta-button:hover {
  transform: translateY(-5px) scale(1.1);
  box-shadow: 0 0 30px rgba(0, 198, 255, 1),
              0 0 60px rgba(0, 198, 255, 0.8),
              0 0 90px rgba(0, 198, 255, 0.6),
              0 10px 40px rgba(0, 0, 0, 0.5),
              inset 0 0 30px rgba(255, 255, 255, 0.3);
}

.outline-button {
  background: transparent;
  color: #fff;
  border: 3px solid #00c6ff;
  padding: 12px 28px;
  border-radius: 30px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 15px;
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 198, 255, 0.3),
              inset 0 0 10px rgba(0, 198, 255, 0.1);
}

.outline-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(0, 198, 255, 0.6), transparent);
  transition: left 0.5s;
}

.outline-button:hover {
  background: rgba(0, 198, 255, 0.2);
  transform: translateY(-5px) scale(1.1);
  box-shadow: 0 0 20px rgba(0, 198, 255, 0.8),
              0 0 40px rgba(0, 198, 255, 0.6),
              0 10px 30px rgba(0, 198, 255, 0.4),
              inset 0 0 20px rgba(0, 198, 255, 0.3);
  text-shadow: 0 0 10px rgba(0, 198, 255, 0.8);
  border-color: #00ffff;
}

.outline-button:hover::before {
  left: 100%;
}

/* 英雄区域 */
.hero {
  min-height: 100vh;
  display: flex;
  align-items: center;
  position: relative;
  perspective: 1000px;
}

.hero-content {
  max-width: 700px;
  z-index: 10;
  animation: heroFadeIn 1.5s ease-out;
  transform-style: preserve-3d;
  transition: transform 0.1s ease-out;
}

@keyframes heroFadeIn {
  from {
    opacity: 0;
    transform: translateX(-50px) rotateY(10deg);
  }
  to {
    opacity: 1;
    transform: translateX(0) rotateY(0);
  }
}

.hero h1 {
  font-size: 3.5rem;
  font-weight: 800;
  margin-bottom: 20px;
  line-height: 1.2;
  background: linear-gradient(90deg, #fff, #00c6ff, #ff00ff, #00c6ff);
  background-size: 200% 100%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: gradientShift 4s ease infinite, glitch 8s infinite;
  position: relative;
  text-shadow: 0 0 30px rgba(0, 198, 255, 0.3);
}

@keyframes glitch {
  0%, 100% { 
    filter: blur(0);
    transform: translate(0);
  }
  98% {
    filter: blur(0);
  }
  99% {
    filter: blur(2px);
    text-shadow: 3px 3px 0 rgba(255, 0, 0, 0.7),
                -3px -3px 0 rgba(0, 255, 255, 0.7);
    transform: translate(3px, -3px);
  }
}

.hero p {
  font-size: 1.2rem;
  margin-bottom: 30px;
  color: #ccc;
  max-width: 600px;
  animation: fadeInUp 1s ease-out 0.5s backwards;
  position: relative;
  text-shadow: 0 0 10px rgba(0, 198, 255, 0.3);
}

.typed-cursor {
  animation: blink 1s infinite;
  color: #00c6ff;
  font-weight: bold;
  text-shadow: 0 0 10px rgba(0, 198, 255, 0.8);
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.hero-buttons {
  display: flex;
  gap: 20px;
  animation: fadeInUp 1s ease-out 0.8s backwards;
}

/* 浮动元素 */
.floating-elements {
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 1;
}

.tech-element {
  position: absolute;
  width: 60px;
  height: 60px;
  border: 3px solid rgba(0, 198, 255, 0.5);
  border-radius: 10px;
  animation: float 20s infinite ease-in-out;
  box-shadow: 0 0 20px rgba(0, 198, 255, 0.4),
              inset 0 0 20px rgba(0, 198, 255, 0.2);
}

.tech-element:nth-child(1) {
  top: 20%;
  left: 10%;
  animation-delay: 0s;
  animation-duration: 15s;
  transform: rotate(45deg);
  border-color: rgba(0, 255, 255, 0.6);
}

.tech-element:nth-child(2) {
  top: 60%;
  left: 80%;
  animation-delay: 5s;
  animation-duration: 20s;
  border-radius: 50%;
  border-color: rgba(255, 0, 255, 0.6);
}

.tech-element:nth-child(3) {
  top: 80%;
  left: 30%;
  animation-delay: 10s;
  animation-duration: 18s;
  border-color: rgba(0, 114, 255, 0.6);
}

@keyframes float {
  0%, 100% {
    transform: translateY(0) rotate(0deg);
    opacity: 0.4;
    box-shadow: 0 0 20px rgba(0, 198, 255, 0.4);
  }
  25% {
    transform: translateY(-30px) rotate(90deg);
    opacity: 0.8;
    box-shadow: 0 0 40px rgba(0, 198, 255, 0.8);
  }
  50% {
    transform: translateY(15px) rotate(180deg);
    opacity: 0.4;
    box-shadow: 0 0 20px rgba(255, 0, 255, 0.4);
  }
  75% {
    transform: translateY(-20px) rotate(270deg);
    opacity: 0.8;
    box-shadow: 0 0 40px rgba(0, 114, 255, 0.8);
  }
}

/* 波纹效果 */
@keyframes ripple {
  to {
    transform: scale(4);
    opacity: 0;
  }
}

/* 响应式 */
@media (max-width: 768px) {
  .tech-website {
    cursor: auto;
  }
  
  #fluidCanvas,
  .cursor-main {
    display: none;
  }
  
  .nav-links {
    display: none;
  }
  
  .hero h1 {
    font-size: 2.5rem;
  }
  
  .hero p {
    font-size: 1rem;
  }
  
  .tech-element {
    display: none;
  }
}

@media (max-width: 576px) {
  .hero-buttons {
    flex-direction: column;
  }
}
</style>