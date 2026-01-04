<template>
  <div class="station-detail-page min-h-screen bg-white dark:bg-black">
    <!-- 加载状态 -->
    <div v-if="isLoading" class="flex justify-center items-center h-96">
      <div class="flex flex-col items-center gap-4">
        <div class="w-12 h-12 border-4 border-ios-blue border-t-transparent rounded-full animate-spin"></div>
        <p class="text-gray-500 dark:text-gray-400">加载中...</p>
      </div>
    </div>

    <!-- 错误状态 -->
    <div v-else-if="error" class="text-center py-20">
      <div class="max-w-md mx-auto px-6">
        <div class="w-20 h-20 bg-red-100 dark:bg-red-900/20 rounded-full flex items-center justify-center mx-auto mb-6">
          <ExclamationTriangleIcon class="w-10 h-10 text-red-500" />
        </div>
        <h2 class="text-2xl font-bold text-gray-900 dark:text-white mb-4">无法加载电台</h2>
        <p class="text-gray-500 dark:text-gray-400 mb-8">{{ error }}</p>
        <button @click="$router.push('/')" class="inline-flex items-center gap-2 bg-ios-blue text-white px-6 py-3 rounded-xl hover:bg-blue-600 transition-all transform hover:scale-105">
          <HomeIcon class="w-5 h-5" />
          返回首页
        </button>
      </div>
    </div>

    <!-- 电台详情内容 -->
    <div v-else-if="station" class="max-w-4xl mx-auto px-4 py-8">
      <!-- 主要信息卡片 -->
      <div class="bg-black dark:bg-gray-800/70 backdrop-blur-xl rounded-3xl shadow-xl border border-white/20 dark:border-gray-700/30 overflow-hidden mb-8 station-card-enter">
        <!-- 大图区域 -->
        <div class="relative h-64 md:h-[450px] flex flex-col items-center justify-center overflow-hidden transition-all duration-1000 animated-background">
          <!-- 动态背景层 -->
          <div class="absolute inset-0 bg-gradient-to-br from-blue-500/30 via-purple-500/30 to-pink-500/30"></div>
          
          <!-- 浮动粒子动画 -->
          <div class="absolute inset-0 overflow-hidden">
            <!-- 大型浮动气泡 -->
            <div class="floating-orb floating-orb-1"></div>
            <div class="floating-orb floating-orb-2"></div>
            <div class="floating-orb floating-orb-3"></div>
            <div class="floating-orb floating-orb-4"></div>
            <div class="floating-orb floating-orb-5"></div>
            <div class="floating-orb floating-orb-6"></div>
            <div class="floating-orb floating-orb-7"></div>
            <div class="floating-orb floating-orb-8"></div>
            
            <!-- 小型粒子 -->
            <div class="particle particle-1"></div>
            <div class="particle particle-2"></div>
            <div class="particle particle-3"></div>
            <div class="particle particle-4"></div>
            <div class="particle particle-5"></div>
            <div class="particle particle-6"></div>
            <div class="particle particle-7"></div>
            <div class="particle particle-8"></div>
          </div>
          
          <!-- 背景装饰 -->
          <div class="absolute inset-0 bg-white/10 dark:bg-black/20"></div>
          
          <!-- 电台图标和唱片效果 -->
          <div class="relative z-10 flex flex-col items-center mt-12 md:mt-[30px]">
            
            <!-- 唱片外圈 -->
            <div class="relative group mb-8">
              <!-- 外层唱片圆盘 -->
              <div class="w-32 h-32 md:w-48 md:h-48 rounded-full bg-gray-800/80 dark:bg-black/80 flex items-center justify-center record-spin shadow-2xl"
                   :class="{ 'animate-spin-slow': isCurrentAndPlaying }">
                <!-- 内层标签区域 -->
                <div class="w-22 h-22 md:w-32 md:h-32 rounded-full bg-white/95 dark:bg-gray-800/95 backdrop-blur-sm flex items-center justify-center shadow-2xl">
                  <img v-if="station.favicon && !faviconError" 
                       :src="station.favicon" 
                       @error="faviconError = true"
                       @load="extractColor"
                       crossorigin="anonymous"
                       class="w-16 h-16 md:w-24 md:h-24 object-cover rounded-full">
                  <img v-else 
                       :src="generatedIconUrl" 
                       :alt="station.name"
                       @load="extractColor"
                       class="w-16 h-16 md:w-24 md:h-24 object-cover rounded-full">
                </div>
                
                <!-- 唱片纹理线条 -->
                <div class="absolute inset-2 rounded-full border border-gray-600/30"></div>
                <div class="absolute inset-4 rounded-full border border-gray-600/20"></div>
                <div class="absolute inset-6 rounded-full border border-gray-600/15"></div>
                <div class="hidden md:block absolute inset-8 rounded-full border border-gray-600/10"></div>
              </div>
            </div>

            <!-- 音频可视化波形图 - 在唱片机下方 -->
            <div class="audio-visualizer-container" ref="visualizerContainer">
              <canvas
                ref="visualizerCanvas"
                :width="canvasWidth"
                :height="canvasHeight"
                class="rainbow-visualizer-canvas"
              ></canvas>
            </div>
          </div>
        </div>

        <!-- 电台信息 -->
        <div class="p-4 md:p-8 bg-white dark:bg-transparent">
          <div class="text-center mb-4 md:mb-8">
            <h1 class="text-xl md:text-4xl font-bold text-gray-900 dark:text-white mb-2 md:mb-3 station-name-slide">{{ station.name }}</h1>
            <p class="text-base md:text-lg text-gray-600 dark:text-gray-300 mb-3 md:mb-4">{{ station.country }}, {{ station.state }}</p>
            
            <!-- 标签 -->
            <div class="flex flex-wrap justify-center gap-1.5 md:gap-2">
              <!-- 其他标签 -->
              <span v-for="(tag, index) in formattedTags.slice(0, 4)" :key="tag"
                    class="px-2 py-1 md:px-4 md:py-2 bg-gradient-to-r from-blue-100 to-purple-100 dark:from-blue-900/30 dark:to-purple-900/30 text-gray-700 dark:text-gray-300 text-xs md:text-sm rounded-full border border-blue-200/50 dark:border-blue-700/50 tag-fade-in"
                    :style="{ animationDelay: `${index * 100}ms` }">
                {{ tag }}
              </span>
            </div>
          </div>

          <!-- 播放控制按钮 -->
          <div class="flex justify-center gap-3 md:gap-4 mb-6 md:mb-8">
            <!-- 主播放按钮 -->
            <button @click="playStation"
                    class="flex items-center gap-2 md:gap-3 px-6 py-3 md:px-8 md:py-4 bg-gradient-to-r from-ios-blue to-blue-600 text-white font-semibold rounded-xl md:rounded-2xl shadow-lg hover:shadow-xl transition-all transform hover:scale-105 active:scale-95 play-button-glow">
              <component :is="playButtonIcon" class="w-5 h-5 md:w-6 md:h-6" />
              <span class="text-sm md:text-lg">{{ isCurrentAndPlaying ? languageStore.t('player.pausePlay') : languageStore.t('player.startPlay') }}</span>
            </button>
          </div>

          <!-- 次要操作按钮 -->
          <div class="flex justify-center gap-4 md:gap-6">
            <button @click="toggleFavorite"
                    class="flex flex-col items-center gap-1 md:gap-2 p-3 md:p-4 rounded-xl md:rounded-2xl transition-all hover:bg-gray-100 dark:hover:bg-gray-700/50 transform hover:scale-105"
                    :class="isFavorite ? 'text-red-500' : 'text-gray-500'">
              <div class="w-10 h-10 md:w-12 md:h-12 rounded-lg md:rounded-xl flex items-center justify-center"
                   :class="isFavorite ? 'bg-red-100 dark:bg-red-900/30' : 'bg-gray-100 dark:bg-gray-700'">
                <HeartIcon class="w-5 h-5 md:w-6 md:h-6" :class="{ 'fill-current': isFavorite }" />
              </div>
              <span class="text-xs md:text-sm font-medium">{{ isFavorite ? languageStore.t('player.favorited') : languageStore.t('player.addToFavorite') }}</span>
            </button>

            <button @click="openShareModal"
                    class="flex flex-col items-center gap-1 md:gap-2 p-3 md:p-4 rounded-xl md:rounded-2xl transition-all hover:bg-gray-100 dark:hover:bg-gray-700/50 transform hover:scale-105 text-gray-500">
              <div class="w-10 h-10 md:w-12 md:h-12 bg-gray-100 dark:bg-gray-700 rounded-lg md:rounded-xl flex items-center justify-center">
                <ShareIcon class="w-5 h-5 md:w-6 md:h-6" />
              </div>
              <span class="text-xs md:text-sm font-medium">{{ languageStore.t('player.share') }}</span>
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 分享模态框 -->
    <ShareModal :visible="isShareModalVisible" :station="station" @close="isShareModalVisible = false" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, watch, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import { useRadioStore } from '@/stores/radio';
import { usePlayerStore } from '@/stores/player';
import { useLanguageStore } from '@/stores/language';
import type { RadioStation } from '@/types/radio';
import { generateIconDataUrl } from '@/utils/iconGenerator';
import { ExclamationTriangleIcon, HomeIcon, PlayIcon, PauseIcon, HeartIcon, ShareIcon } from '@heroicons/vue/24/outline';
import ShareModal from '@/components/ShareModal.vue';

const route = useRoute();
const radioStore = useRadioStore();
const playerStore = usePlayerStore();
const languageStore = useLanguageStore();

const station = ref<RadioStation | null>(null);
const isLoading = ref(true);
const error = ref<string | null>(null);
const isShareModalVisible = ref(false);
const faviconError = ref(false);
const dynamicBackgroundStyle = ref({});

// 音频可视化相关
const visualizerContainer = ref<HTMLDivElement>();
const visualizerCanvas = ref<HTMLCanvasElement>();
const canvasWidth = ref(320);
const canvasHeight = ref(120);
let animationId: number | null = null;

const stationUuid = computed(() => route.params.uuid as string);

const isCurrentAndPlaying = computed(() => {
  return playerStore.currentStation?.stationuuid === station.value?.stationuuid && playerStore.isPlaying;
});

const playButtonIcon = computed(() => {
  return isCurrentAndPlaying.value ? PauseIcon : PlayIcon;
});

const isFavorite = computed(() => {
  if (!station.value) return false;
  return playerStore.favorites.some(fav => fav.stationuuid === station.value!.stationuuid);
});

const formattedTags = computed(() => {
  if (!station.value?.tags) return [];
  return station.value.tags.split(',').map(tag => tag.trim()).filter(Boolean).slice(0, 5);
});

const generatedIconUrl = computed(() => {
  return station.value ? generateIconDataUrl(station.value.name) : '';
});

// 简单的颜色提取函数
const extractColor = (event: Event) => {
  const img = event.target as HTMLImageElement;
  if (!img) return;

  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  canvas.width = img.width;
  canvas.height = img.height;
  
  try {
    ctx.drawImage(img, 0, 0, img.width, img.height);
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    
    let r = 0, g = 0, b = 0;
    let count = 0;
    
    // 采样图像中心区域的像素
    for (let i = 0; i < data.length; i += 16) { // 每隔4个像素采样
      r += data[i];
      g += data[i + 1];
      b += data[i + 2];
      count++;
    }
    
    r = Math.floor(r / count);
    g = Math.floor(g / count);
    b = Math.floor(b / count);
    
    // 创建基于主色调的动态背景
    const primaryColor = `rgba(${r}, ${g}, ${b}, 0.6)`;
    const secondaryColor = `rgba(${r + 30}, ${g + 20}, ${b + 40}, 0.4)`;
    const tertiaryColor = `rgba(${r - 20}, ${g + 40}, ${b + 30}, 0.3)`;
    
    dynamicBackgroundStyle.value = {
      background: `linear-gradient(135deg, ${primaryColor}, ${secondaryColor}, ${tertiaryColor})`,
      transition: 'background 2s ease-in-out'
    };
  } catch (e) {
    console.log('无法提取颜色，使用默认背景');
    // 使用默认的iOS风格背景
    dynamicBackgroundStyle.value = {
      background: 'linear-gradient(135deg, rgba(59, 130, 246, 0.6), rgba(139, 92, 246, 0.4), rgba(236, 72, 153, 0.3))',
      transition: 'background 2s ease-in-out'
    };
  }
};

const playStation = () => {
  if (!station.value) return;
  if (isCurrentAndPlaying.value) {
    playerStore.pauseStation();
  } else {
    playerStore.playStation(station.value);
  }
};

const toggleFavorite = () => {
  if (!station.value) return;
  playerStore.toggleFavorite(station.value);
};

const openShareModal = () => {
  isShareModalVisible.value = true;
};

// 初始化音频可视化 - Siri Wave 风格
const initAudioVisualizer = () => {
  // 简单地启动波形动画，无需复杂的音频上下文
  // 这种方式更稳定，跨域兼容性更好，且视觉效果更现代
  startSiriWave();
};

// Siri Wave 变量
let phase = 0;
// 定义三条波浪的颜色和参数
const waves = [
  { color: 'rgba(59, 130, 246, 0.6)', speed: 0.01, amplitude: 0.5 }, // 蓝色
  { color: 'rgba(236, 72, 153, 0.6)', speed: 0.02, amplitude: 0.4 }, // 粉色
  { color: 'rgba(139, 92, 246, 0.6)', speed: 0.015, amplitude: 0.3 }, // 紫色
];

const startSiriWave = () => {
  if (!visualizerCanvas.value) return;
  const canvas = visualizerCanvas.value;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  const draw = () => {
    // 检查画布是否还存在
    if (!visualizerCanvas.value) {
      if (animationId) cancelAnimationFrame(animationId);
      return;
    }

    const width = canvas.width;
    const height = canvas.height;
    
    ctx.clearRect(0, 0, width, height);
    ctx.globalCompositeOperation = 'screen'; // 混合模式产生发光效果

    const baseAmplitude = height * 0.35;
    const isPlaying = isCurrentAndPlaying.value;
    const t = Date.now() * 0.002;

    // 绘制每一条波浪
    waves.forEach((wave, index) => {
      ctx.beginPath();
      ctx.strokeStyle = wave.color;
      ctx.lineWidth = 2;
      
      // 根据播放状态调整振幅
      let currentAmp = wave.amplitude;
      if (isPlaying) {
        // 播放时：使用正弦波模拟动态起伏
        // 加入一些随机性让它看起来更像真实的音频响应
        currentAmp *= (1 + Math.sin(t + index * 2) * 0.4 + Math.cos(t * 0.5) * 0.2);
      } else {
        // 暂停时：保持微小的呼吸感
        currentAmp *= 0.15;
      }

      // 绘制波形路径
      for (let x = 0; x <= width; x += 2) {
        // 归一化 X 坐标 (-2 到 2) 用于高斯衰减计算
        const scaledX = (x / width) * 4 - 2;
        
        // 高斯窗口函数：使波形在两端自然衰减为0
        const attenuation = Math.exp(-Math.pow(scaledX, 2));
        
        // 波形公式：y = 振幅 * sin(频率 * x + 相位) * 衰减
        const y = height / 2 + 
                  Math.sin(x * 0.01 + phase * wave.speed + index + t) * 
                  baseAmplitude * currentAmp * attenuation;
        
        if (x === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.stroke();
    });
    
    // 更新相位
    phase += isPlaying ? 2 : 0.5;
    
    // 继续下一帧
    animationId = requestAnimationFrame(draw);
  };
  
  // 启动动画循环
  if (animationId) cancelAnimationFrame(animationId);
  draw();
};

// 停止可视化
const stopVisualizer = () => {
  if (animationId) {
    cancelAnimationFrame(animationId);
    animationId = null;
  }
};

// 更新画布尺寸
const updateCanvasSize = () => {
  if (!visualizerContainer.value) return;
  
  const containerWidth = visualizerContainer.value.offsetWidth;
  // 增加宽度和高度以获得更好的视觉效果
  canvasWidth.value = Math.min(800, containerWidth > 0 ? containerWidth : 320);
  canvasHeight.value = 120; // 增加高度，使波形更舒展
  
  // 重新启动动画以适应新尺寸
  stopVisualizer();
  setTimeout(startSiriWave, 50);
};

// 监听播放状态变化
watch(isCurrentAndPlaying, () => {
  // 播放状态改变时，动画循环会自动调整振幅，无需重启
});

onMounted(async () => {
  try {
    const result = await radioStore.getStationByUuid(stationUuid.value);
    if (result) {
      station.value = result;
      // Update page title
      document.title = `${result.name} | 全球电台`;
    } else {
      throw new Error('未找到该电台。');
    }
  } catch (e: any) {
    error.value = e.message;
  } finally {
    isLoading.value = false;
  }

  // 初始化可视化
  await nextTick();
  updateCanvasSize();
  
  // 启动 Siri Wave 动画
  setTimeout(() => {
    initAudioVisualizer();
  }, 100);
  
  // 监听窗口大小变化
  window.addEventListener('resize', updateCanvasSize);
});

onUnmounted(() => {
  stopVisualizer();
  window.removeEventListener('resize', updateCanvasSize);
});
</script>

<style scoped>
.station-card-enter {
  animation: slideUp 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.station-name-slide {
  animation: slideInFromLeft 1s cubic-bezier(0.25, 0.46, 0.45, 0.94) 0.3s both;
}

.tag-fade-in {
  animation: fadeInUp 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

.play-button-glow {
  box-shadow: 0 0 20px 0 rgba(59, 130, 246, 0.5);
}

.record-spin {
  transition: transform 0.3s ease;
}

.animate-spin-slow {
  animation: spin 12s linear infinite;
}

/* 动态背景效果 */
.animated-background {
  background: linear-gradient(45deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
}

@media (min-width: 768px) { /* md断点及以上 */
  .animated-background {
    background: linear-gradient(45deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
    background-size: 400% 400%;
    animation: gradientShift 15s ease infinite;
  }
}

/* 浮动球体 */
.floating-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(1px);
  animation: float 20s infinite linear;
}

.floating-orb-1 {
  width: 180px;
  height: 180px;
  background: radial-gradient(circle, rgba(255, 107, 107, 0.4) 0%, rgba(255, 107, 107, 0) 70%);
  top: 5%;
  left: 5%;
  animation-duration: 20s;
  animation-delay: -3s;
}

.floating-orb-2 {
  width: 120px;
  height: 120px;
  background: radial-gradient(circle, rgba(78, 205, 196, 0.5) 0%, rgba(78, 205, 196, 0) 70%);
  top: 70%;
  right: 10%;
  animation-duration: 25s;
  animation-delay: -8s;
}

.floating-orb-3 {
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(199, 121, 208, 0.3) 0%, rgba(199, 121, 208, 0) 70%);
  bottom: 10%;
  left: 15%;
  animation-duration: 30s;
  animation-delay: -12s;
}

.floating-orb-4 {
  width: 80px;
  height: 80px;
  background: radial-gradient(circle, rgba(255, 195, 113, 0.6) 0%, rgba(255, 195, 113, 0) 70%);
  top: 40%;
  right: 30%;
  animation-duration: 18s;
  animation-delay: -5s;
}

.floating-orb-5 {
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, rgba(129, 236, 236, 0.4) 0%, rgba(129, 236, 236, 0) 70%);
  bottom: 50%;
  right: 5%;
  animation-duration: 22s;
  animation-delay: -10s;
}

.floating-orb-6 {
  width: 100px;
  height: 100px;
  background: radial-gradient(circle, rgba(255, 159, 243, 0.5) 0%, rgba(255, 159, 243, 0) 70%);
  top: 15%;
  left: 40%;
  animation-duration: 27s;
  animation-delay: -6s;
}

.floating-orb-7 {
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, rgba(108, 92, 231, 0.7) 0%, rgba(108, 92, 231, 0) 70%);
  top: 60%;
  left: 70%;
  animation-duration: 16s;
  animation-delay: -2s;
}

.floating-orb-8 {
  width: 220px;
  height: 220px;
  background: radial-gradient(circle, rgba(52, 211, 153, 0.3) 0%, rgba(52, 211, 153, 0) 70%);
  top: 20%;
  right: 50%;
  animation-duration: 35s;
  animation-delay: -15s;
}

/* 小粒子 */
.particle {
  position: absolute;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  animation: particleFloat 15s infinite linear;
}

.particle-1 { width: 8px; height: 8px; top: 15%; left: 25%; animation-delay: -2s; animation-duration: 18s; }
.particle-2 { width: 12px; height: 12px; top: 45%; left: 75%; animation-delay: -6s; animation-duration: 22s; }
.particle-3 { width: 6px; height: 6px; top: 75%; left: 15%; animation-delay: -4s; animation-duration: 16s; }
.particle-4 { width: 15px; height: 15px; top: 25%; right: 20%; animation-delay: -8s; animation-duration: 20s; }
.particle-5 { width: 4px; height: 4px; bottom: 35%; left: 60%; animation-delay: -3s; animation-duration: 19s; }
.particle-6 { width: 10px; height: 10px; top: 65%; right: 35%; animation-delay: -7s; animation-duration: 17s; }
.particle-7 { width: 7px; height: 7px; bottom: 15%; right: 50%; animation-delay: -1s; animation-duration: 21s; }
.particle-8 { width: 14px; height: 14px; top: 85%; left: 40%; animation-delay: -9s; animation-duration: 23s; }

/* 音频可视化容器 */
.audio-visualizer-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: -2rem; 
  margin-bottom: 1.5rem;
  position: relative;
  z-index: 20;
  width: 100%;
}

.rainbow-visualizer-canvas {
  background: transparent;
  border-radius: 8px;
  /* 移除阴影以获得更干净的发光效果 */
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInFromLeft {
  from {
    opacity: 0;
    transform: translateX(-30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@keyframes float {
  0%, 100% {
    transform: translateY(0px) translateX(0px) scale(1);
    opacity: 0.7;
  }
  25% {
    transform: translateY(-20px) translateX(10px) scale(1.1);
    opacity: 0.9;
  }
  50% {
    transform: translateY(-10px) translateX(-15px) scale(0.9);
    opacity: 0.5;
  }
  75% {
    transform: translateY(-30px) translateX(5px) scale(1.05);
    opacity: 0.8;
  }
}

@keyframes particleFloat {
  0% {
    transform: translateY(0px) translateX(0px);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    transform: translateY(-100px) translateX(50px);
    opacity: 0;
  }
}

.station-detail-container {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 40vh; /* 缩减动态背景高度，优化底部间距 */
  padding: 2rem;
  overflow: hidden;
  box-sizing: border-box;
}

.player-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex-grow: 1;
  width: 100%;
}
</style>
