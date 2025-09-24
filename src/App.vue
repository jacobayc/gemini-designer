<template>
  <div id="app">
    <header>
    </header>
    <main>
      <ChatWithEnv ref="chatWithEnvRef" @headlines-updated="handleHeadlinesUpdated" />
      <NewsHeadlines :headlines="newsData" />
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import ChatWithEnv from './components/ChatWithEnv.vue';
import NewsHeadlines from './components/NewsHeadlines.vue';

const chatWithEnvRef = ref(null);
const newsData = ref([]);

const handleHeadlinesUpdated = (headlines) => {
  // 1. 뉴스 데이터 상태 업데이트
  newsData.value = headlines;
  // 2. 받아온 데이터를 세션 스토리지에 저장
  sessionStorage.setItem('ai-news-headlines', JSON.stringify(headlines));
};

onMounted(() => {
  const storedHeadlines = sessionStorage.getItem('ai-news-headlines');
  if (storedHeadlines) {
    // 세션 스토리지에 데이터가 있으면 그것을 사용
    newsData.value = JSON.parse(storedHeadlines);
  } else {
    // 데이터가 없으면 API 호출
    const today = new Date();
    const year = today.getFullYear();
    const month = today.getMonth() + 1;
    const day = today.getDate();
    const formattedDate = `${year}년 ${month}월 ${day}일`;
    chatWithEnvRef.value?.runExamplePrompt(`${formattedDate} 뉴스 헤드라인 알려줘`);
  }
});
</script>

<style>
/* 전역 스타일을 여기에 추가할 수 있습니다. */
body {
  background-color: var(--backgroundColor, #fff);
  transition: background-color 0.3s;
}
</style>

