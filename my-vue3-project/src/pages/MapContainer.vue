<template>
  <div id="container"></div>
</template>

<script setup>
import { onMounted, onUnmounted } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

let map = null;
let geolocation = null;

onMounted(() => {
  // 配置安全密钥
  window._AMapSecurityConfig = {
    securityJsCode: "d8c7e8f1e4b3a2c1d9e8f7a6b5c4d3e2", // 请替换为你自己的安全密钥
  };
  
  AMapLoader.load({
    key: "8543c182e1fdf2e2a508de70ebed62ea", // 你的Web端开发者Key
    version: "2.0",
    plugins: [
      "AMap.Scale",
      "AMap.Geolocation",
      "AMap.Marker",
      "AMap.InfoWindow"
    ],
  })
    .then((AMap) => {
      // 初始化地图
      map = new AMap.Map("container", {
        viewMode: "3D",
        zoom: 16,
        center: [116.397428, 39.90923],
      });

      // 添加比例尺
      const scale = new AMap.Scale();
      map.addControl(scale);

      // 获取当前位置
      geolocation = new AMap.Geolocation({
        enableHighAccuracy: true, // 是否使用高精度定位，默认:true
        timeout: 10000, // 超过10秒后停止定位，默认：5s
        buttonPosition: 'RB', // 定位按钮的停靠位置
        buttonOffset: new AMap.Pixel(10, 20), // 定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
        zoomToAccuracy: true, // 定位成功后是否自动调整地图视野到定位点
      });

      map.addControl(geolocation);
      
      // 自动获取当前位置
      geolocation.getCurrentPosition((status, result) => {
        if (status === 'complete') {
          onComplete(result);
        } else {
          onError(result);
        }
      });

      function onComplete(data) {
        console.log('定位成功：', data);
        // 添加当前位置标记
        const marker = new AMap.Marker({
          position: [data.position.lng, data.position.lat],
          title: '我的位置'
        });
        map.add(marker);
        
        // 添加信息窗体
        const infoWindow = new AMap.InfoWindow({
          content: `<div style="padding: 10px;">当前位置<br/>经度：${data.position.lng.toFixed(6)}<br/>纬度：${data.position.lat.toFixed(6)}</div>`,
          offset: new AMap.Pixel(0, -30)
        });
        marker.on('click', () => {
          infoWindow.open(map, marker.getPosition());
        });
      }

      function onError(data) {
        console.error('定位失败：', data);
        alert('定位失败，请检查是否开启定位权限');
      }
    })
    .catch((e) => {
      console.error('地图加载失败：', e);
    });
});

onUnmounted(() => {
  map?.destroy();
});
</script>

<style scoped>
#container {
  padding: 0px;
  margin: 0px;
  width: 100%;
  height: 100%;
}
</style>
