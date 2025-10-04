<template>
  <div>
    <section class="toolbar" role="search" aria-label="搜索活动">
      <label class="sr-only" for="kw">搜索活动</label>
      <input id="kw" v-model="keyword" placeholder="输入关键字过滤…" />
    </section>
    
    <main class="content">
      <section class="summary" aria-live="polite">
        共 <strong>{{ filtered.length }}</strong> 个活动
      </section>
      
      <section class="grid" aria-label="活动列表">
        <article
          v-for="a in filtered"
          :key="a.id"
          class="card"
          :class="{ signed: a.signed, shaking: a.shaking }"
          tabindex="0"
          :aria-labelledby="'tit-'+a.id"
        >
          <header class="head">
            <h3 :id="'tit-'+a.id" class="title">{{ a.title }}</h3>
            <span v-if="a.signed" class="badge" aria-label="已报名">已报名</span>
          </header>
          <p class="meta"><span>{{ a.time }}</span> · <span>{{ a.place }}</span></p>
          <p class="desc">{{ a.desc }}</p>
          <div class="ops">
            <button class="btn" type="button" @click="toggleSignup(a.id)">
              {{ a.signed ? '取消' : '报名' }}
            </button>
          </div>
        </article>
      </section>
    </main>
    
    <!-- 弹窗提示 -->
    <div v-if="showToast" class="toast" role="alert" aria-live="assertive">
      {{ toastMessage }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'ActivityList',
  setup() {
    const keyword = ref('');
    const acts = ref([
      { id:1, title:'校园摄影行', time:'10-20 14:00', place:'图书馆广场', desc:'基础构图练习' },
      { id:2, title:'羽毛球友谊赛', time:'10-25 19:00', place:'体育馆A2', desc:'自带球拍' },
      { id:3, title:'演讲训练营', time:'10-28 18:30', place:'学活101', desc:'结构化表达' }
    ]);
    const showToast = ref(false);
    const toastMessage = ref('');

    const filtered = computed(() => {
      const kw = keyword.value.trim().toLowerCase();
      return !kw
        ? acts.value
        : acts.value.filter(a =>
            a.title.toLowerCase().includes(kw) ||
            a.place.toLowerCase().includes(kw) ||
            a.desc.toLowerCase().includes(kw)
          );
    });
    
    function toggleSignup(id) {
      const it = acts.value.find(a => a.id === id);
      if (it) {
        const wasSigned = it.signed;
        it.signed = !it.signed;
        
        if (!wasSigned && it.signed) {
          it.shaking = true;
          setTimeout(() => {
            it.shaking = false;
          }, 600);
          
          toastMessage.value = `已成功报名「${it.title}」`;
          showToast.value = true;
          setTimeout(() => {
            showToast.value = false;
          }, 3000);
        }
      }
    }
    
    return { keyword, filtered, toggleSignup, showToast, toastMessage };
  }
}
</script>