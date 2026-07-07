<script setup lang="ts">
    import { onMounted, ref } from 'vue';

    const btcPrice = ref('0');
    const ethPrice = ref('0');
    const adaPrice = ref('0');

    let intervalId: number;

    const timer = ref(5);

    const setTimer = (value: number) => {
        localStorage.setItem('timer', value.toString());

        clearInterval(intervalId);
        intervalId = setInterval(() => {
            handleSpeak();
        }, 1000 * 60 * value);
    };

    onMounted(() => {
        const value = localStorage.getItem('timer');
        const valueInt = parseInt(value || '5');
        timer.value = valueInt;
        if (value) {
            setTimer(valueInt);
        }
    });

    const BINANCE_WS_URL =
        'wss://stream.binance.com:9443/stream?streams=btcusdt@ticker/ethusdt@ticker/adausdt@ticker';

    const ws = new WebSocket(`${BINANCE_WS_URL}`);

    const handleSpeak = () => {
        const text = `比特幣${btcPrice.value}元, 以太幣${ethPrice.value}元, 艾達幣${adaPrice.value}元`;

        const utterance = new SpeechSynthesisUtterance(text);
        utterance.lang = 'zh-TW'; // 設定語言為繁體中文
        utterance.rate = 0.8; // 設定語速

        const voices = window.speechSynthesis.getVoices();

        const voice = voices.find((v) => v.lang === 'zh-TW'); // 選擇正確的語音

        if (voice) utterance.voice = voice;

        window.speechSynthesis.speak(utterance);
    };


    ws.onmessage = function (msg) {
        const parsedData = JSON.parse(msg.data);

        // 獲取 ticker 的數據
        const stream = parsedData.stream; // e.g., "btcusdt@ticker" 或 "ethusdt@ticker"
        const tickerData = parsedData.data;

        if (stream === 'btcusdt@ticker') {
            btcPrice.value = parseInt(tickerData.c).toString(); // 最新成交價 (current price)
        }

        if (stream === 'ethusdt@ticker') {
            ethPrice.value = parseInt(tickerData.c).toString(); // 最新成交價 (current price)
        }

        if (stream === 'adausdt@ticker') {
            adaPrice.value = parseFloat(tickerData.c).toString(); // 最新成交價 (current price)
        }
    };
</script>

<template>
    <button
        id="speakButton"
        @click="handleSpeak"
        style="color: white"
    >
        開始播放價格
    </button>
    <div class="card">
        <div>比特幣價格：{{ btcPrice }} USDT</div>
        <div>以太幣價格：{{ ethPrice }} USDT</div>
        <div>艾達幣價格：{{ adaPrice }} USDT</div>
    </div>
    <div>
        <input
            type="number"
            v-model="timer"
            class="loop-input"
            @change="
                (event) => {
                    const elemnet = event.target as HTMLInputElement;

                    setTimer(parseInt(elemnet.value));
                }
            "
        />
        分鐘播放一次
    </div>
</template>

<style scoped>
.loop-input{
    width: 50px;
    text-align: center;
}
</style>
