<script setup lang="ts">
  import { ref } from 'vue';
  // import WebSocket from ws

  defineProps<{ msg: string }>();

  const count = ref(0);
  const BINANCE_WS_URL = 'wss://stream.binance.com:9443/ws';
  const symbol = 'btcusdt';
  const stream = `${symbol}@ticker`;

  const ws = new WebSocket(`${BINANCE_WS_URL}/${stream}`);

  const handleSpeak = () => {
    const utterance = new SpeechSynthesisUtterance(count.value.toString());
    utterance.lang = 'zh-TW'; // 設定語言為繁體中文
    const voices = window.speechSynthesis.getVoices();
    const voice = voices.find((v) => v.lang === 'zh-TW'); // 選擇正確的語音
    if (voice) utterance.voice = voice;

    console.log(utterance, voice);

    window.speechSynthesis.speak(utterance);
  };

  setInterval(() => {
    console.log('?');

    handleSpeak();
  }, 5000);

  ws.onmessage = function (msg) {
    const ticker = JSON.parse(msg.data);
    const price = ticker.c; // 最新成交價 (current price)
    console.log(`[${new Date().toLocaleTimeString()}] BTC/USDT Price: $${price}`);

    count.value = parseInt(price);
  };
</script>

<template>
  <button
    id="speakButton"
    @click="handleSpeak"
    style="color: white"
  >
    開始播放比特幣價格
  </button>
  <div class="card">
    <div type="button">比特幣價格：{{ count }} USDT</div>
  </div>
</template>

<style scoped>
  .read-the-docs {
    color: #888;
  }
</style>
