<template>
  <div class="weather" v-if="weatherData.ipinfo.city && weatherData.weather.weather">
    <span>{{ weatherData.ipinfo.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{ weatherData.weather.winddirection }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script setup>
import { getIpInfofromIpapi, getWeatherfromQweather } from "@/api";
import { Error } from "@icon-park/vue-next";

// 和风天气开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  ipinfo: {
    city: null, // 城市
    longitude: null, // 经度
    latitude: null, // 纬度
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
    // 从 ip-api.com 获取用户 IP 的位置信息
    const ipInfo = await getIpInfofromIpapi();
    if (!ipInfo || ipInfo.status !== "success") {
      throw "IP定位失败";
    }

    // 更新 ipinfo 数据
    weatherData.ipinfo.city = ipInfo.city;
    weatherData.ipinfo.longitude = ipInfo.lon;
    weatherData.ipinfo.latitude = ipInfo.lat;

    // 从 qweather 获取天气信息
    const lon = parseFloat(ipInfo.lon).toFixed(2);
    const lat = parseFloat(ipInfo.lat).toFixed(2);
    const weatherResult = await getWeatherfromQweather(mainKey, lon, lat);
    if (!weatherResult || weatherResult.code !== "200") {
      throw "天气信息获取失败";
    }

    // 更新 weather 数据
    const now = weatherResult.now;
    weatherData.weather = {
      weather: now.text,
      temperature: now.temp,
      winddirection: now.windDir,
      windpower: now.windScale + "级",
    };
  } catch (error) {
    console.error("天气信息加载失败:" + error);
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

onMounted(() => {
  // 调用获取天气
  getWeatherData();
});
</script>
