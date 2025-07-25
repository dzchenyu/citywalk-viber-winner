<template>
  <view id="container"></view>
</template>

<script>
import { onMounted, onUnmounted, ref, watch } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

export default {
  props: {
    startNavigation: {
      type: Boolean,
      default: false
    },
    regenerateRoute: {
      type: Boolean,
      default: false
    }
  },
  emits: ['navigation-ready', 'route-generated'],
  setup(props, { emit }) {
    let map = null;
    let geolocation = null;
    let walking = null;
    let currentRoute = null;
    let currentPositionMarker = null;
    let startMarker = null;
    let endMarker = null;
    let routePolyline = null;
    let pathMarkers = [];

    onMounted(() => {
      console.log('MapContainer 组件挂载，开始初始化地图...');
      
      // 配置安全密钥
      window._AMapSecurityConfig = {
        securityJsCode: "aca55b5cc6a9a9aabb99860e12d4bffe",
      };
      
      AMapLoader.load({
        key: "8543c182e1fdf2e2a508de70ebed62ea",
        version: "2.0",
        plugins: [
          "AMap.Scale",
          "AMap.Geolocation",
          "AMap.Marker",
          "AMap.InfoWindow",
          "AMap.Walking",
          "AMap.Polyline"
        ],
      })
        .then((AMap) => {
          console.log('高德地图API加载成功');
          
          // 初始化地图
          map = new AMap.Map("container", {
            viewMode: "3D",
            zoom: 16,
            center: [116.397428, 39.90923], // 默认中心点
          });

          // 添加比例尺
          const scale = new AMap.Scale({
            position: 'LB',
            offset: new AMap.Pixel(10, 60) // 向上移动，避免挡住路线卡片
          });
          map.addControl(scale);

          // 创建定位控件
          geolocation = new AMap.Geolocation({
            enableHighAccuracy: true,
            timeout: 10000,
            buttonPosition: 'RB',
            buttonOffset: new AMap.Pixel(10, 100), // 向上移动，避免重叠
            zoomToAccuracy: true,
            showButton: true,
            showMarker: true,
            showCircle: true
          });

          map.addControl(geolocation);

          // 完全隐藏高德地图商标和版权信息
          map.on('complete', function() {
            setTimeout(() => {
              const logoElements = document.querySelectorAll('.amap-logo, .amap-copyright');
              logoElements.forEach(el => {
                if (el) el.style.display = 'none';
              });
            }, 100);
          });

          // 创建步行导航实例
          walking = new AMap.Walking({
            map: map,
            hideMarkers: false,
            isOutline: true,
            outlineColor: '#ffeeee',
            autoFitView: true
          });

          // 立即开始定位
          initLocation();

          function initLocation() {
            console.log('开始获取当前位置...');
            
            // 使用高德地图定位
            geolocation.getCurrentPosition((status, result) => {
              console.log('定位状态:', status, '结果:', result);
              
              if (status === 'complete') {
                const position = [result.position.lng, result.position.lat];
                console.log('定位成功:', position);
                handleLocationSuccess(position, result);
              } else {
                console.log('定位失败，使用浏览器API');
                useBrowserLocation();
              }
            });
          }

          function useBrowserLocation() {
            if (navigator.geolocation) {
              navigator.geolocation.getCurrentPosition(
                (position) => {
                  const coords = [position.coords.longitude, position.coords.latitude];
                  console.log('浏览器定位成功:', coords);
                  handleLocationSuccess(coords, {
                    position: { lng: coords[0], lat: coords[1] },
                    message: '浏览器定位'
                  });
                },
                (error) => {
                  console.log('浏览器定位失败:', error);
                  handleLocationSuccess([116.397428, 39.90923], {
                    position: { lng: 116.397428, lat: 39.90923 },
                    message: '默认位置'
                  });
                },
                {
                  enableHighAccuracy: true,
                  timeout: 8000,
                  maximumAge: 0
                }
              );
            } else {
              console.log('浏览器不支持定位，使用默认位置');
              handleLocationSuccess([116.397428, 39.90923], {
                position: { lng: 116.397428, lat: 39.90923 },
                message: '默认位置'
              });
            }
          }

          function handleLocationSuccess(position, result) {
            console.log('最终位置:', position, '来源:', result.message || '高德定位');
            
            // 更新地图中心到当前位置
            map.setCenter(position);
            map.setZoom(16);

            // 移除蓝色当前位置标记，只保留绿色起点标记
            // 不再创建 currentPositionMarker

            // 生成2公里内步行路线
            generateWalkingRoute(position);
          }

          // 基于当前位置生成2公里内步行路线
          function generateWalkingRoute(startPosition) {
            console.log('正在从当前位置生成2公里内步行路线:', startPosition);
            
            // 计算500米-1公里范围内的随机目的地
            // 1度约等于111公里，所以500米=0.0045度，1公里=0.009度
            const angle = Math.random() * 2 * Math.PI;
            const distance = 0.0045 + Math.random() * 0.0045; // 0.0045-0.009度约500米-1公里
            const endPosition = [
              startPosition[0] + distance * Math.cos(angle),
              startPosition[1] + distance * Math.sin(angle)
            ];

            const route = {
              name: "当前位置1-1.5公里步行路线",
              start: startPosition,
              end: endPosition,
              distance: "1-1.5公里",
              actualDistance: "计算中...",
              duration: "计算中..."
            };

            currentRoute = route;

            // 清除之前的标记和路线
            if (startMarker) map.remove(startMarker);
            if (endMarker) map.remove(endMarker);
            if (routePolyline) map.remove(routePolyline);
            // 清除路径点标记
            pathMarkers.forEach(marker => map.remove(marker));
            pathMarkers = [];
            
            // 添加起点标记（当前位置）
            startMarker = new AMap.Marker({
              position: route.start,
              title: '起点 (我的位置)',
              icon: new AMap.Icon({
                size: new AMap.Size(25, 34),
                image: '//a.amap.com/jsapi_demos/static/demo-center/icons/dir-start-marker.png',
                imageSize: new AMap.Size(135, 40),
                imageOffset: new AMap.Pixel(-9, -3)
              })
            });

            // 添加终点标记
            endMarker = new AMap.Marker({
              position: route.end,
              title: '1-1.5公里目的地',
              icon: new AMap.Icon({
                size: new AMap.Size(25, 34),
                image: '//a.amap.com/jsapi_demos/static/demo-center/icons/dir-end-marker.png',
                imageSize: new AMap.Size(135, 40),
                imageOffset: new AMap.Pixel(-95, -3)
              })
            });

            map.add([startMarker, endMarker]);

            // 使用步行导航生成完整路线
            walking.search(route.start, route.end, (status, result) => {
              console.log('步行导航结果:', status, result);
              
              if (status === 'complete') {
                const path = result.routes[0].path;
                const distance = result.routes[0].distance;
                const duration = result.routes[0].time;
                
                // 更新实际距离和时间
                route.actualDistance = (distance / 1000).toFixed(1) + '公里';
                route.duration = Math.ceil(duration / 60) + '分钟';

                // 创建路线折线
                routePolyline = new AMap.Polyline({
                  path: path,
                  isOutline: true,
                  outlineColor: '#ffeeee',
                  borderWeight: 2,
                  strokeColor: "#3366FF",
                  strokeOpacity: 0.9,
                  strokeWeight: 6,
                  strokeStyle: "solid",
                  lineJoin: 'round',
                  lineCap: 'round',
                  zIndex: 50,
                });
                
                map.add(routePolyline);
                
                // 添加路径点标记
                addPathMarkers(path);
                
                map.setFitView([routePolyline, startMarker, endMarker, ...pathMarkers]);

                console.log('2公里内路线生成完成:', route);
                emit('route-generated', route);
                emit('navigation-ready', result);
              } else {
                console.log('步行导航失败，使用备用方案');
                drawFallbackRoute(route);
              }
            });
          }

          // 备用方案：绘制直线
          function drawFallbackRoute(route) {
            const path = [route.start, route.end];
            routePolyline = new AMap.Polyline({
              path: path,
              isOutline: true,
              outlineColor: '#ffeeee',
              borderWeight: 2,
              strokeColor: "#FF6B6B",
              strokeOpacity: 0.8,
              strokeWeight: 5,
              strokeStyle: "dashed",
              strokeDasharray: [10, 5],
              lineJoin: 'round',
              lineCap: 'round',
              zIndex: 50,
            });
            
            map.add(routePolyline);
            map.setFitView([routePolyline, startMarker, endMarker]);
            
            route.actualDistance = "约2公里";
            route.duration = "约25分钟";
            emit('route-generated', route);
          }

          // 开始导航
          function startNavigation() {
            if (currentRoute && walking) {
              console.log('开始2公里内导航:', currentRoute);
              
              // 清除之前的路线
              if (routePolyline) map.remove(routePolyline);

              // 重新添加标记（只添加起点和终点标记）
              map.add([startMarker, endMarker]);

              // 开始导航
              walking.search(currentRoute.start, currentRoute.end, (status, result) => {
                if (status === 'complete') {
                  console.log('2公里内导航开始');
                  
                  // 添加导航提示
                  const infoWindow = new AMap.InfoWindow({
                    content: `<div style="padding: 10px; font-size: 14px;">
                      <h4 style="margin: 0 0 10px 0;">🚶‍♂️ 开始2公里内步行导航</h4>
                      <p style="margin: 5px 0;"><strong>起点:</strong> 我的当前位置</p>
                      <p style="margin: 5px 0;"><strong>终点:</strong> 2公里内目的地</p>
                      <p style="margin: 5px 0;"><strong>实际距离:</strong> ${currentRoute.actualDistance}</p>
                      <p style="margin: 5px 0;"><strong>预计时间:</strong> ${currentRoute.duration}</p>
                      <p style="margin: 5px 0;"><strong>提示:</strong> 请沿蓝色路线行走</p>
                    </div>`,
                    offset: new AMap.Pixel(0, -30)
                  });
                  
                  infoWindow.open(map, currentRoute.end);
                }
              });
            }
          }

          // 添加路径点标记
          function addPathMarkers(path) {
            if (!path || path.length < 3) return;
            
            // 计算路径点间隔，每200-300米添加一个标记
            const totalDistance = AMap.GeometryUtil.distanceOfLine(path);
            const interval = 250; // 250米间隔
            const stepCount = Math.floor(totalDistance / interval);
            
            if (stepCount < 2) return;
            
            const step = Math.floor(path.length / (stepCount + 1));
            
            for (let i = 1; i <= stepCount; i++) {
              const index = Math.min(i * step, path.length - 2);
              if (index > 0 && index < path.length - 1) {
                const marker = new AMap.Marker({
                  position: [path[index].lng, path[index].lat],
                  title: `路径点 ${i}`,
                  icon: new AMap.Icon({
                    size: new AMap.Size(12, 12),
                    image: 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIiIGhlaWdodD0iMTIiIHZpZXdCb3g9IjAgMCAxMiAxMiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPGNpcmNsZSBjeD0iNiIgY3k9IjYiIHI9IjQiIGZpbGw9IiMzMzY2RkYiIHN0cm9rZT0id2hpdGUiIHN0cm9rZS13aWR0aD0iMSIvPgo8L3N2Zz4K',
                    imageSize: new AMap.Size(12, 12)
                  }),
                  zIndex: 80
                });
                pathMarkers.push(marker);
                map.add(marker);
              }
            }
            
            console.log(`已添加 ${pathMarkers.length} 个路径点标记`);
          }

          // 重新生成路线的方法
          function regenerateRoute() {
            console.log('重新生成路线...');
            // 使用新的随机角度和距离
            if (currentRoute && currentRoute.start) {
              const startPosition = currentRoute.start;
              
              // 计算新的随机目的地
              const angle = Math.random() * 2 * Math.PI;
              const distance = 0.0045 + Math.random() * 0.0045; // 0.0045-0.009度约500米-1公里
              const endPosition = [
                startPosition[0] + distance * Math.cos(angle),
                startPosition[1] + distance * Math.sin(angle)
              ];

              // 清除之前的路线
              if (routePolyline) map.remove(routePolyline);
              if (startMarker) map.remove(startMarker);
              if (endMarker) map.remove(endMarker);
              pathMarkers.forEach(marker => map.remove(marker));
              pathMarkers = [];

              // 重新生成路线
              generateWalkingRoute(startPosition);
            } else {
              // 如果没有当前位置，重新定位
              initLocation();
            }
          }

          // 监听props变化，触发导航
          watch(() => props.startNavigation, (newVal) => {
            if (newVal) {
              startNavigation();
            }
          });

          // 监听重新生成路线
          watch(() => props.regenerateRoute, (newVal) => {
            if (newVal) {
              regenerateRoute();
            }
          });

          onUnmounted(() => {
            if (map) map.destroy();
          });

          return {
            startNavigation,
            regenerateRoute
          };
        })
        .catch((e) => {
          console.error('地图加载失败：', e);
        });
    });

    return {
      startNavigation,
      regenerateRoute
    };
  }
}
</script>

<style scoped>
#container {
  padding: 0px;
  margin: 0px;
  width: 100%;
  height: 100%;
  min-height: 400px;
}

/* 强制隐藏高德地图商标和版权信息 */
:deep(.amap-logo),
:deep(.amap-copyright) {
  display: none !important;
  opacity: 0 !important;
  visibility: hidden !important;
}
</style>
