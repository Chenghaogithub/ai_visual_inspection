<script setup>
import { computed, onBeforeUnmount, ref } from 'vue';

const progress = ref(0);
const isScanning = ref(false);
const hasResult = ref(false);
const statusText = ref('等待光学神经同步');
const logItems = ref([
  '视觉引擎已休眠，等待人类授权',
  '摄像头沙盒接口预热完成',
  '虹膜谐振模型处于低功耗待命'
]);
const faceX = ref(46);
const faceY = ref(35);
const eyePulse = ref(0);

let progressTimer;
let logTimer;
let faceTimer;
let finishTimer;

const statusClass = computed(() => {
  if (hasResult.value) return 'online';
  if (isScanning.value) return 'busy';
  return 'idle';
});

const logs = [
  '正在调用摄像头...',
  '神经网络矩阵计算中...',
  '正在深度学习'
];

function clearTimers() {
  [progressTimer, logTimer, faceTimer, finishTimer].forEach((timer) => {
    if (timer) window.clearInterval(timer);
  });
  window.clearTimeout(finishTimer);
}

function pushLog(text) {
  logItems.value = [text, ...logItems.value].slice(0, 11);
}

function startScan() {
  if (isScanning.value) return;

  clearTimers();
  progress.value = 0;
  isScanning.value = true;
  hasResult.value = false;
  statusText.value = '正在深度学习';
  logItems.value = ['正在深度学习'];

  let logIndex = 0;
  progressTimer = window.setInterval(() => {
    if (progress.value < 82) {
      progress.value += Math.floor(Math.random() * 8) + 4;
    } else if (progress.value < 99) {
      progress.value += 1;
    }

    progress.value = Math.min(progress.value, 99);

    if (progress.value > 18) statusText.value = '神经网络矩阵计算中';
    if (progress.value > 45) statusText.value = '正在深度学习';
    if (progress.value > 72) statusText.value = '正在深度学习';

    if (progress.value === 99) {
      window.clearInterval(progressTimer);
      statusText.value = '正在深度学习';
      finishTimer = window.setTimeout(() => {
        progress.value = 100;
        statusText.value = '检测完成';
        isScanning.value = false;
        hasResult.value = true;
        window.clearInterval(logTimer);
        window.clearInterval(faceTimer);
        pushLog('高级视觉推理完成：眼睛状态已被发现');
      }, 2000);
    }
  }, 230);

  logTimer = window.setInterval(() => {
    pushLog(logs[logIndex % logs.length]);
    logIndex += 1;
  }, 520);

  faceTimer = window.setInterval(() => {
    faceX.value = 39 + Math.sin(Date.now() / 530) * 8;
    faceY.value = 31 + Math.cos(Date.now() / 620) * 5;
    eyePulse.value = (eyePulse.value + 1) % 4;
  }, 120);
}

function closeResult() {
  hasResult.value = false;
  statusText.value = '等待下一次睁眼奇迹';
}

onBeforeUnmount(clearTimers);
</script>

<template>
  <main class="app-shell">
    <section class="vision-console" aria-label="人工智能视觉检测仪">
      <div class="console-topbar">
        <div>
          <p class="eyebrow">CYBER LOW-POWER OPTICS LAB</p>
          <h1>人工智能视觉检测</h1>
        </div>
        <div class="system-state" :class="statusClass">
          <span></span>
          {{ statusText }}
        </div>
      </div>

      <div class="console-grid">
        <div class="camera-panel">
          <div class="camera-feed" :class="{ scanning: isScanning }">
            <div class="scanlines"></div>
            <div class="noise"></div>
            <div class="reticle"></div>
            <div v-if="isScanning" class="gaze-hint">请双眼注视屏幕</div>
            <div
              class="face-lock"
              :style="{ left: `${faceX}%`, top: `${faceY}%` }"
            >
              <span class="corner tl"></span>
              <span class="corner tr"></span>
              <span class="corner bl"></span>
              <span class="corner br"></span>
              <div class="eye-line" :class="`pulse-${eyePulse}`"></div>
            </div>
            <div class="camera-copy">
              <span>CAM-01</span>
              <strong>{{ isScanning ? 'FACE TRACKING' : 'READY' }}</strong>
            </div>
          </div>
        </div>

        <aside class="matrix-panel">
          <div class="panel-title">
            <span>LIVE MATRIX</span>
            <b>{{ progress }}%</b>
          </div>
          <div class="matrix-stream">
            <p v-for="(item, index) in logItems" :key="`${item}-${index}`">
              <span>{{ String(index + 1).padStart(2, '0') }}</span>
              {{ item }}
            </p>
          </div>
        </aside>
      </div>

      <div class="control-strip">
        <button class="primary-action" :disabled="isScanning" @click="startScan">
          {{ isScanning ? '检测运行中' : '开始检测' }}
        </button>
        <div class="progress-shell" aria-label="检测进度">
          <div class="progress-bar" :style="{ width: `${progress}%` }"></div>
        </div>
      </div>
    </section>

    <div v-if="hasResult" class="result-backdrop" role="presentation">
      <section class="result-dialog" role="dialog" aria-modal="true" aria-label="检测结果">
        <p>检测结果</p>
        <h2>您的眼睛目前处于<br /><strong>“睁开”</strong>状态。</h2>
        <button @click="closeResult">收到</button>
      </section>
    </div>
  </main>
</template>
