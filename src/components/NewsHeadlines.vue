<template>
  <div class="news-headlines-container">
    <!-- Detail View -->
    <div v-if="selectedHeadline">
      <button @click="backToList" class="back-button">&larr; 목록으로</button>
      <h4>{{ selectedHeadline.categories }}</h4>
      <h3>{{ selectedHeadline.title }}</h3>
      <p class="content-area">{{ selectedHeadline.contents }}</p>
    </div>
    <!-- List View -->
    <div v-else>
      <h3>오늘의 간단 뉴스</h3>
      <div v-if="hasHeadlines">
        <div v-for="(news, category) in groupedHeadlines" :key="category" class="category-group">
          <h4>{{ category }}</h4>
          <ul>
            <li v-for="headline in news" :key="headline.id" @click="showDetails(headline)">
              {{ headline.title }}
            </li>
          </ul>
        </div>
      </div>
      <div v-else>
        <p>헤드라인을 불러오는 중입니다...</p>
      </div>
  </div>
</div>
</template>

<script setup>
import { ref, computed  } from 'vue';

const props = defineProps({
  headlines: {
    type: Array,
    default: () => []
  }
});

const selectedHeadline = ref(null);

const hasHeadlines = computed(() => props.headlines && props.headlines.length > 0);

const groupedHeadlines = computed(() => {
  if (!hasHeadlines.value) return {};
  return props.headlines.reduce((acc, headline) => {
    const category = headline.categories || '기타';
    if (!acc[category]) {
      acc[category] = [];
    }
    acc[category].push(headline);
    return acc;
  }, {});
});

const showDetails = (headline) => {
  selectedHeadline.value = headline;
};

const backToList = () => {
  selectedHeadline.value = null;
};
</script>

<style scoped>
.news-headlines-container {
  font-family: Arial, sans-serif;
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  margin: 20px;
  background-color: var(--containerBackgroundColor, #ffffff);
  color: var(--color, #333);
  transition: background-color 0.3s, color 0.3s;
}

h3 {
  margin-top: 0;
  border-bottom: 2px solid #eee;
  padding-bottom: 10px;
  color: var(--color, #333);
  border-bottom-color: var(--chatAreaBackgroundColor, #eee);
}

h4 {
  color: var(--color, #555);
  margin-top: 20px;
  margin-bottom: 10px;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

li {
  padding: 12px 15px;
  border-bottom: 1px solid var(--chatAreaBackgroundColor, #f0f0f0);
  cursor: pointer;
  transition: background-color 0.2s;
}


li:hover {
  background-color: var(--chatAreaBackgroundColor, #f9f9f9);
}

/* Detail View Styles */
.back-button {
  background: none;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 20px;
  font-size: 14px;
  transition: background-color 0.2s;
  color: var(--color, #333);
  border: 1px solid var(--color, #ccc);
}

.back-button:hover {
  background-color: var(--chatAreaBackgroundColor, #f0f0f0);
}

.content-area {
  line-height: 1.7;
  white-space: pre-wrap; /* 줄바꿈과 공백을 유지 */
  padding: 15px;
  border-radius: 5px;
  color: var(--chatAreaColor, #444);
  background-color: var(--chatAreaBackgroundColor, #fdfdfd);
  border: 1px solid var(--chatAreaBackgroundColor, #eee);
}

.category-group + .category-group {
  margin-top: 20px;
}
</style>

