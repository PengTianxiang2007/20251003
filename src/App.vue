<template>
  <div id="app" class="shell">
    <header class="site-header">
      <div class="header-left">
        <h1 class="title-part title-simple">Simple</h1>
      </div>
      <div class="logo-container">
        <img alt="Vue logo" class="logo" src="https://cn.vuejs.org/images/logo.svg" />
      </div>
      <div class="header-right">
        <h1 class="title-part title-activities">Activities</h1>
      </div>
    </header>
    <p class="subtitle">仅改HTML+CSS,JS不动</p>
    
    <!-- 活动列表组件 -->
    <ActivityList />
    
    <footer class="site-footer">
      <small>© 2025 Your Class</small>
    </footer>
  </div>
</template>

<script>
import ActivityList from './ActivityList.vue'

export default {
  name: 'App',
  components: {
    ActivityList
  }
}
</script>

<style>
/* 基础样式调整 - 确保页脚在最底端 */
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
body {
  display: flex;
  flex-direction: column;
  background: linear-gradient(-45deg, #e0f2fe, #dbeafe, #e0e7ff, #faf5ff);
  background-size: 400% 400%;
  animation: gradientBG 15s ease infinite;
}

@keyframes gradientBG {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.shell {
  flex: 1;
  margin: 0 auto;
  width: 100%;
  max-width: 1400px;
  padding: 20px 24px 0;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

/* --------------- 头部样式 --------------- */
:root {
  --color-border: #e5e7eb;
  --color-bg: #fff;
  --gap: 12px;            
  --radius: 36px;          
  --shadow: 0 8px 24px rgba(0,0,0,.06);
}
.site-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 4px;
  width: 100%;
}
.header-left, .header-right {
  flex: 1;
  text-align: center;
}
.logo-container {
  flex: 0 0 auto;
}
.logo {
  display: block;
  margin: 0 auto;
  border-radius: 50%;
  padding: 8px;
  background: var(--color-bg);
  box-shadow: var(--shadow);
  width: 120px;
  height: 120px;
}
.title-part {
  margin: 0;
  line-height: 1.1;
  font-weight: 700;
  font-size: clamp(2.5rem, 6vw, 5rem);
  letter-spacing: .2px;
  animation: 0.5s linear infinite;
}
.title-simple {
  animation-name: colorScrollSimple;
}
@keyframes colorScrollSimple {
  0% { color: #3b82f6; }
  25% { color: #8b5cf6; }
  50% { color: #ec4899; }
  75% { color: #6366f1; }
  100% { color: #3b82f6; }
}
.title-activities {
  animation-name: colorScrollActivities;
}
@keyframes colorScrollActivities {
  0% { color: #10b981; }
  25% { color: #06b6d4; }
  50% { color: #22d3ee; }
  75% { color: #14b8a6; }
  100% { color: #10b981; }
}
/* 副标题样式 */
.subtitle {
  margin: 8px 0 10px;
  font-size: 0.9rem;
  opacity: .75;
  text-align: center;
}

/* --------------- 工具栏样式 --------------- */
.toolbar {
  margin-bottom: calc(var(--gap) + 5px);
  background: var(--color-bg, #fff);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 16px;
  display: flex;
  justify-content: center;
  margin-top: 8px;
}
.toolbar input {
  width: min(800px, 100%);
  height: 50px;
  padding: 0 28px;
  font-size: 1.1rem;
  border: 1px solid var(--color-border);
  border-radius: 30px;
  outline: none;
  text-align: center;
}
.toolbar input:focus {
  box-shadow: 0 0 0 4px color-mix(in oklab, #3b82f6 25%, transparent);
}

/* --------------- 主要内容区域 --------------- */
.content { 
  flex: 1;
  margin-bottom: 8px;
  display: flex;
  flex-direction: column;
}
.summary {
  text-align: center;
  font-size: 16px;          
  opacity: .7;
  margin: 0 2px 16px;
  padding: 10px 0;
}
.grid {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  height: 100%;
  min-height: 500px;
  margin-top: auto;
}
.card {
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  padding: 30px 34px;
  background: var(--color-bg, #fff);
  box-shadow: 0 6px 20px rgba(0,0,0,.05);
  outline: none;
  transition: transform .1s ease, box-shadow .1s ease, background 2s ease-in-out;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
  box-sizing: border-box;
}
.card:hover { transform: translateY(-4px); box-shadow: 0 12px 30px rgba(0,0,0,.08); }
.card:focus { box-shadow: 0 0 0 4px color-mix(in oklab, #10b981 28%, transparent), 0 6px 20px rgba(0,0,0,.05); }

/* 增强抖动动画 */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-10px); }
  20%, 40%, 60%, 80% { transform: translateX(10px); }
}
/* 应用抖动动画的类 */
.card.shaking {
  animation: shake 0.6s ease-in-out;
}

.head { 
  display:flex; 
  align-items:center; 
  gap:12px; 
  margin-bottom: 16px; 
  justify-content: center; 
  flex-wrap: wrap; 
  padding: 10px 0;
}
.card .title { 
  font-size: clamp(2.5rem, 8vw, 3.5rem);
  margin: 15px 0;
  line-height: 1.3; 
  text-align: center;
  font-weight: 700;
  width: 100%;
}
.badge {
  font-size: 16px;
  padding: 8px 18px;
  border: 1px solid var(--color-border);
  border-radius: 999px;
  background: color-mix(in oklab, #22c55e 18%, #ffffff 84%);
  margin-top: 8px;
}
.meta { 
  font-size: 17px;
  opacity:.75; 
  margin: 12px 0 -7px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--color-border);
  text-align: center;
}
.desc { 
  margin: 16px 0 1.5rem;
  line-height:1.8; 
  font-size: 1.70rem;
  flex: 1;
  text-align: center;
  max-width: 90%;
  margin-left: auto;
  margin-right: auto;
}
.ops { 
  display:flex; 
  justify-content:center; 
  margin-top: -20px; /* 修复：将过度上移的-150px调整为合理的-20px */
  padding: 5px 0; 
  margin-bottom: 10px; 
}
.btn {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  font-size: 2rem;
  border: 2px solid #3b82f6;
  background: white;
  color: #3b82f6;
  cursor: pointer;
  transition: all .2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}
.btn:hover {
  background: #3b82f6;
  color: white;
  transform: translateY(-4px) scale(1.05);
  box-shadow: 0 6px 12px rgba(59, 130, 246, 0.2);
}
/* 已报名状态的按钮样式 */
.card.signed .btn {
  border-color: #ef4444;
  color: #ef4444;
}
.card.signed .btn:hover {
  background: #ef4444;
  color: white;
  box-shadow: 0 6px 12px rgba(239, 68, 68, 0.2);
}
.card.signed { background: color-mix(in oklab, var(--color-bg) 55%, #22c55e 38%); }

/* --------------- 弹窗样式 --------------- */
.toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.85);
  color: white;
  padding: 30px 40px; /* 增加内边距使弹窗更大 */
  border-radius: 20px; /* 增加圆角 */
  font-size: 1.5rem; /* 增大字体 */
  z-index: 1000;
  animation: fadeInOut 3s ease-in-out;
  text-align: center;
  max-width: 400px; /* 增加最大宽度 */
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.5);
  border: 3px solid rgba(255, 255, 255, 0.3);
  font-weight: 700;
}

@keyframes fadeInOut {
  0%, 100% { opacity: 0; transform: translate(-50%, -50%) scale(0.8); }
  10%, 90% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}

/* --------------- 页脚样式 --------------- */
.site-footer { 
  text-align: center; 
  opacity: .7; 
  font-size: 14px;
  padding: 15px 0;
  margin-top: 0;
  width: 100%;
  box-sizing: border-box;
}
.sr-only {
  position:absolute; width:1px; height:1px; padding:0; margin:-1px;
  overflow:hidden; clip:rect(0,0,0,0); border:0;
}

/* 响应式调整 */
@media (max-width: 1200px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
  .title-part { font-size: clamp(2rem, 5vw, 3.5rem); }
  .logo {
    width: 100px;
    height: 100px;
  }
  .card {
    padding: 24px 20px;
  }
  .card .title {
    font-size: clamp(2.2rem, 7vw, 2.8rem);
  }
  .desc {
    font-size: 1.15rem;
  }
  .btn {
    width: 70px;
    height: 70px;
    font-size: 0.9rem;
  }
  .ops {
    margin-top: -10px; /* 移动端按钮位置微调 */
  }
  .toast {
    font-size: 1.3rem; /* 移动端字体也相应增大 */
    padding: 25px 30px; /* 移动端内边距也相应增大 */
    max-width: 320px; /* 移动端最大宽度也相应增大 */
  }
}
@media (max-width: 640px) {
  .shell { padding: 16px 16px 0; }
  .toolbar { padding: 10px; }
  .toolbar input { 
    height: 44px; 
    border-radius: 26px;
  }
  .site-header {
    flex-direction: column;
    gap: 12px;
  }
  .header-left, .header-right {
    width: 100%;
  }
}
</style>