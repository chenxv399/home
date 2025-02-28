<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
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
import { getWeather, getWeatherfromWeatherapi, getAdcodefromAmap } from "@/api";
import { Error } from "@icon-park/vue-next";

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;
const WeatherapiKey = import.meta.env.VITE_WEATHERAPI_KEY;

// 天气数据
const weatherData = reactive({
  adCode: {
    city: null, // 城市
    adcode: null, // 城市编码
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
    // 使用weatherapi获取天气和地理信息
    const WeatherapiResult = await getWeatherfromWeatherapi(WeatherapiKey);
    if (WeatherapiResult.location.country == "China"){
      //使用高德逆地理编码获取Adcode
      const jingdu = WeatherapiResult.location.lon;
      const weidu = WeatherapiResult.location.lat;
      const adCode = await getAdcodefromAmap(mainkey,jingdu,weidu);
      console.log(adCode);
      if (adCode.infocode !== "10000") {
        throw "地区查询失败";
      }
      weatherData.adCode = {
        city: adCode.regeocode.addressComponent.city,
        adcode: adCode.regeocode.addressComponent.adcode,
      };
      // 获取高德天气信息
      const result = await getWeather(mainKey, weatherData.adCode.adcode);
      weatherData.weather = {
        weather: result.lives[0].weather,
        temperature: result.lives[0].temperature,
        winddirection: result.lives[0].winddirection?.endsWith("风")
          ? result.lives[0].winddirection
          : result.lives[0].winddirection + "风",
        windpower: result.lives[0].windpower + "级",
      }; 
    }else{
      weatherData.adCode = {
        city: WeatherapiResult.location.name,
        adcode: 000000,
      };

      weatherData.weather = {
        weather: WeatherapiResult.current.condition.text,
        temperature: WeatherapiResult.current.temp_c,
        winddirection: WeatherapiResult.current.wind_dir,
        windpower: WeatherapiResult.current.wind_kph + "km/h",
      };
    }
  } catch (error) {
    console.error("天气信息获取失败:" + error);
    onError("天气信息获取失败");
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
