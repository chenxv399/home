<template>
  <div class="weather-container" v-if="weatherData.location.city && weatherData.weather.weather">
    <div class="weather" ref="weatherRef">
      <div
        class="weather-content"
        :class="{ 'weather-scroll': isOverflow, 'weather-center': !isOverflow }"
        :style="isOverflow ? weatherScrollStyle : {}"
        ref="contentRef"
      >
        <span>{{ weatherData.location.city }}&nbsp;</span>
        <span>{{ weatherData.weather.weather }}&nbsp;</span>
        <span>{{ weatherData.weather.temperature }}℃</span>
        <span class="sm-hidden">
          &nbsp;{{ weatherData.weather.winddirection }}&nbsp;
        </span>
        <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
      </div>
    </div>
  </div>
  <div class="weather-container" v-else>
    <div class="weather weather-center-fail">
      <span>天气数据获取失败</span>
    </div>
  </div>
</template>

<script setup>
import { getWeatherinfo } from "@/api";
import { Error } from "@icon-park/vue-next";
import { ref, reactive, onMounted, nextTick, watch } from "vue";

// 天气 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 引用天气元素
const weatherRef = ref(null);
const isOverflow = ref(false);
// 滚动动画样式
const weatherScrollStyle = ref({});
// 内容元素引用
const contentRef = ref(null);

// 天气数据
const weatherData = reactive({
  location: {
    city: null, // 城市
  },
  weather: {
    weather: null, // 天气现象
    temperature: null, // 实时气温
    winddirection: null, // 风向描述
    windpower: null, // 风力级别
  },
});

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 获取天气信息
    const weatherResult = await getWeatherinfo(mainKey);
    if (!weatherResult.ok) {
      throw new Error(weatherResult.error);
    }

    // 更新天气数据
    weatherData.location.city = weatherResult.location.city;
    weatherData.weather.weather = weatherResult.weather.text;
    weatherData.weather.temperature = weatherResult.weather.temp;
    weatherData.weather.winddirection = weatherResult.weather.windDir;
    weatherData.weather.windpower = weatherResult.weather.windScale;

  } catch (error) {
    console.error("天气信息加载失败:", error);
    onError("天气信息加载失败");
  }
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  console.error(message);
};

// 检查内容是否超出容器
const checkOverflow = async () => {
  await nextTick();
  if (weatherRef.value && contentRef.value) {
    const weatherEl = weatherRef.value;
    const contentEl = contentRef.value;
    // 判断是否超出
    isOverflow.value = contentEl.scrollWidth > weatherEl.clientWidth;
    if (isOverflow.value) {
      // 计算滚动距离
      const distance = contentEl.scrollWidth - weatherEl.clientWidth;
      // 动画时长可根据距离自适应
      const duration = Math.max(2, (distance / 100) * 2);
      // 生成唯一keyframes名称
      const keyframesName = `weather-scroll-keyframes`;
      // 移除旧的keyframes
      const oldStyle = document.getElementById('weather-scroll-keyframes-style');
      if (oldStyle) oldStyle.remove();
      // 动态插入 keyframes
      const style = document.createElement('style');
      style.id = 'weather-scroll-keyframes-style';
      style.innerHTML = `
        @keyframes ${keyframesName} {
          0% { transform: translateX(0); }
          90% { transform: translateX(-${distance}px); }
          100% { transform: translateX(0); }
        }
      `;
      document.head.appendChild(style);
      // 设置动画样式
      weatherScrollStyle.value = {
        animation: `${keyframesName} ${duration}s linear infinite`
      };
    } else {
      weatherScrollStyle.value = {};
    }
  }
};

onMounted(() => {
  getWeatherData().then(() => {
    // 获取天气数据后检查是否需要滚动
    checkOverflow();
  });

  // 监听窗口大小变化，重新检查是否需要滚动
  window.addEventListener('resize', checkOverflow);
});

// 监听天气数据变化，重新检测滚动
watch(
  () => [
    weatherData.location.city,
    weatherData.weather.weather,
    weatherData.weather.temperature,
    weatherData.weather.winddirection,
    weatherData.weather.windpower
  ],
  () => {
    checkOverflow();
  }
);
</script>

<style scoped>
.weather-container {
  width: 100%;
  overflow: hidden;
}

.weather {
  position: relative;
  width: 100%;
  overflow: hidden;
  white-space: nowrap;
}

.weather-content {
  display: flex;
  align-items: center;
  white-space: nowrap;
  width: max-content;
}

.weather-center {
  justify-content: center;
  width: 100%;
  text-align: center;
  margin: 0 auto;
}

.weather-center-fail {
  text-align: center;
}

.weather-scroll {
  will-change: transform;
}

@media (min-width: 910px) and (max-width: 1200px) {
  .sm-hidden {
    display: none;
  }
}
</style>
