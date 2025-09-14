<template>
  <div class="gemini-chat-container">
    <div class="chat-area">
      <div v-if="isUiChange" class="response-box">
        <p>당신의 요청에 맞춰 디자인을 변경했습니다!</p>
      </div>
      <div v-else class="response-box">
        <p>UI 변경 요청이 아닙니다. UI/UX 변경을 위한 질문을 해주세요.</p>
      </div>
      <!-- <div v-else class="placeholder-text">
        <p>무엇을 도와드릴까요?</p>
      </div> -->
    </div>

    <div class="input-area">
      <input
        v-model="prompt"
        @keyup.enter="callGeminiAPI"
        placeholder="디자인 변경을 요청해보세요..."
        :disabled="isLoading"
      />
      <button @click="callGeminiAPI" :disabled="isLoading">
        <span v-if="isLoading">변경중...</span>
        <span v-else>보내기</span>
      </button>
    </div>

    <div class="example-prompts">
      <p>ex : 이렇게 요청해보세요:</p>
      <ul>
        <li v-for="example in examplePrompts" :key="example">
          <button @click="runExamplePrompt(example)" :disabled="isLoading">{{ example }}</button>
        </li>
      </ul>
    </div>
    
    <div v-if="error" class="error-message">
      <p>오류가 발생했습니다: {{ error }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

// 환경 변수에서 API 키와 엔드포인트를 가져옵니다.
const GEMINI_API_KEY = import.meta.env.VITE_GEMINI_API_KEY;
const GEMINI_API_ENDPOINT = import.meta.env.VITE_GEMINI_API_ENDPOINT;

// Vue 3의 반응형 상태 변수들
const prompt = ref('');
const response = ref(null);
const isLoading = ref(false);
const error = ref(null);

// UI 변경 응답을 관리하기 위한 변수
const uiResponse = ref(null); 
const isUiChange = ref(false);

const examplePrompts = [
  '어두운 다크 모드로 변경해줘',
  '바다처럼 시원한 느낌으로 UI를 변경해줘',
  '따뜻한 오렌지 계열 색상으로 바꿔줘',
  '숲속처럼 편안한 녹색 테마로 만들어줘',
  '배경은 연노랑에 버튼은 핑크색으로',
  '전체적으로 남보라 분위기에 버튼은 눈에 띄게'
];

const runExamplePrompt = (example) => {
  if (isLoading.value) return;
  prompt.value = example;
  callGeminiAPI();
};

// Gemini API를 호출하는 함수
const callGeminiAPI = async () => {
  if (!prompt.value.trim() || !GEMINI_API_KEY) {
    if (!GEMINI_API_KEY) {
      error.value = "API 키가 로드되지 않았습니다. .env 파일을 확인해주세요.";
    }
    return;
  }

  isLoading.value = true;
  error.value = null;

  try {
    const res = await axios.post(`${GEMINI_API_ENDPOINT}?key=${GEMINI_API_KEY}`, {
      contents: [{
        parts: [{
          // AI에게 JSON 응답을 요청하는 시스템 프롬프트
          text: `
            사용자의 요청에 따라 웹사이트의 스타일을 변경하는 JSON 객체를 생성하세요.
            요청이 UI 변경과 관련 없을 경우, "UI 변경 요청이 아닙니다. UI/UX 변경을 위한 질문을 해주세요."라고 응답하세요.
            
            JSON 형식 예시:
             {"type": "ui-change", "data": {"backgroundColor": "#...", "color": "#...", "containerBackgroundColor": "#...", "chatAreaBackgroundColor": "#...", "chatAreaColor": "#...", "buttonBackgroundColor": "#...", "buttonColor": "#...", "buttonHoverBackgroundColor": "#..."}}

            사용자 요청: ${prompt.value}
          `
        }]
      }]
    });
    
    const geminiText = res.data.candidates[0].content.parts[0].text.trim();

    try {
      // Gemini 응답에서 JSON 블록을 추출합니다.
    // 응답이 ```json ... ``` 형식으로 감싸져 있거나, 다른 텍스트와 함께 올 수 있습니다.
    const jsonMatch = geminiText.match(/\{[\s\S]*\}/);

    if (jsonMatch && jsonMatch[0]) {
      const jsonString = jsonMatch[0];
      const parsed = JSON.parse(jsonString);
      
        if (parsed.type === 'ui-change' && parsed.data) {
          isUiChange.value = true;
          uiResponse.value = parsed.data;
          response.value = null; // UI 변경 완료 메시지는 템플릿에서 직접 처리
          applyStyles(uiResponse.value); // 스타일 적용 함수 호출
        } else {
          // JSON 형식은 맞지만 'ui-change' 타입이 아닐 경우
          isUiChange.value = false;
          response.value = geminiText;
        }
      } else {
         // 응답에서 JSON 객체를 찾지 못한 경우, 일반 텍스트로 처리
        isUiChange.value = false;
        response.value = geminiText;
      }
    } catch (parseError) {
      // JSON 파싱에 실패하면 일반 텍스트 응답으로 간주
      isUiChange.value = false;
      response.value = geminiText;
      console.error("JSON 파싱 오류:", parseError, "원본 텍스트:", geminiText);
    }
    
  } catch (err) {
    console.error("API 호출 오류:", err);
    error.value = "API 호출 중 문제가 발생했습니다.";
    response.value = null;
  } finally {
    isLoading.value = false;
    prompt.value = ''; // 성공 또는 실패 후 입력 필드 초기화
  }
};

// JSON 응답을 받아 CSS 변수(Custom Properties)에 적용하는 함수
const applyStyles = (styles) => {
  const rootElement = document.documentElement; // HTML의 :root 선택
  for (const key in styles) {
    if (styles[key]) {
      // --backgroundColor, --color 등으로 CSS 변수 설정
      rootElement.style.setProperty(`--${key}`, styles[key]);
    }
  }
};
</script>

<style scoped>
/* 이전 예제와 동일한 스타일 */
.gemini-chat-container {
  display: flex;
  flex-direction: column;
  max-width: 1200px;
  width: 90%;
  margin: 40px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-family: Arial, sans-serif;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  /* background-color: #f9f9f9; */
  /* background: orange; */
  background-color: var(--containerBackgroundColor, #f9f9f9); /* 변수 적용, 기본값 설정 */
  color: var(--color, #333); /* 변수 적용, 기본값 설정 */
}

h2 {
  text-align: center;
  color: #333;
}

.chat-area {
  min-height: 50px;
  padding: 15px;
  border: 1px solid #eee;
  border-radius: 6px;
  background-color: var(--chatAreaBackgroundColor, #fff); /* 변수 적용 */
  margin-bottom: 20px;
  word-wrap: break-word;
  white-space: pre-wrap;
  word-break: break-word;
}

.response-box p {
  line-height: 1.6;
  color: var(--chatAreaColor, #555); /* 변수 적용 */
}

.placeholder-text {
  color: var(--chatAreaColor, #aaa); /* 변수 적용 */
  text-align: center;
  margin-top: 50px;
}

.input-area {
  display: flex;
  gap: 10px;
}

input[type="text"] {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

input {
  width: 60%;
  flex-grow: 1;
}

button {
  min-width: 96px;
  padding: 10px 20px;
  background-color: var(--buttonBackgroundColor, #42b983);
  color: var(--buttonColor, #fff);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover:not(:disabled) {
  background-color: var(--buttonHoverBackgroundColor, #369c67);
}

button:disabled {
  background-color: #a0a0a0;
  cursor: not-allowed;
}

.error-message {
  color: #e53935;
  margin-top: 10px;
  text-align: center;
}

.example-prompts {
  margin-top: 20px;
  text-align: left;
}

.example-prompts p {
  font-weight: bold;
  color: #555;
  margin-bottom: 10px;
  font-size: 0.9em;
}

.example-prompts ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.example-prompts button {
  background-color: #f0f0f0;
  color: #333;
  border: 1px solid #ddd;
  padding: 8px 12px;
  border-radius: 20px; /* rounded buttons */
  font-size: 0.9em;
}

.example-prompts button:hover:not(:disabled) {
  background-color: #e0e0e0;
  border-color: #ccc;
}
</style>