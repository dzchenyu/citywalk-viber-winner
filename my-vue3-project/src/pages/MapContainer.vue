<template>
  <view id="container">
    <!-- 导航指引UI -->
    <view class="navigation-guide" v-if="isNavigating">
      <view class="guide-content">
        <view class="direction-icon">
          {{ currentDirection.icon }}
        </view>
        <view class="guide-info">
          <text class="guide-instruction">{{ currentDirection.instruction }}</text>
          <text class="guide-distance">{{ currentDirection.distance }}</text>
        </view>
      </view>
      <view class="progress-bar">
        <view class="progress" :style="{ width: navigationProgress + '%' }"></view>
      </view>
    </view>
  </view>
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
    },
    currentRouteIndex: {
      type: Number,
      default: 0
    },
    routes: {
      type: Array,
      default: () => []
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
    let navigationWatcher = null;
    let currentStepIndex = 0;
    let navigationSteps = [];
    
    // 导航状态
    const isNavigating = ref(false);
    const navigationProgress = ref(0);
    const currentDirection = ref({
      icon: '↑',
      instruction: '准备导航...',
      distance: '计算中...'
    });

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
            pitch:50,
            buildingAnimation: true,
            terrain:true,
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
            showMarker: false, // 不显示默认标记，我们将创建自定义标记
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

            // 创建当前位置标记
            if (currentPositionMarker) {
              map.remove(currentPositionMarker);
            }
            
            currentPositionMarker = new AMap.Marker({
              position: position,
              icon: new AMap.Icon({
                size: new AMap.Size(24, 24),
                image: 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPGNpcmNsZSBjeD0iMTIiIGN5PSIxMiIgcj0iMTAiIGZpbGw9IiMwMDY2RkYiIGZpbGwtb3BhY2l0eT0iMC44Ii8+CjxjaXJjbGUgY3g9IjEyIiBjeT0iMTIiIHI9IjYiIGZpbGw9IndoaXRlIi8+CjxjaXJjbGUgY3g9IjEyIiBjeT0iMTIiIHI9IjMiIGZpbGw9IiMwMDY2RkYiLz4KPC9zdmc+',
                imageSize: new AMap.Size(24, 24)
              }),
              zIndex: 100,
              angle: result.heading || 0 // 使用定位返回的方向，如果有的话
            });
            
            map.add(currentPositionMarker);

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

            // 清除之前的路线
            if (routePolyline) map.remove(routePolyline);
            
            // 只显示路径线，不添加任何标记

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
                
                // 只显示路径线
                map.setFitView([routePolyline]);

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
            map.setFitView([routePolyline]);
            
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

              // 开始导航
              walking.search(currentRoute.start, currentRoute.end, (status, result) => {
                if (status === 'complete') {
                  console.log('2公里内导航开始');
                  
                  // 获取路径和导航信息
                  const path = result.routes[0].path;
                  const steps = result.routes[0].steps;
                  
                  // 创建导航指引UI
                  createNavigationUI(steps);
                  
                  // 开始实时位置追踪
                  startRealTimeNavigation(path);
                }
              });
            }
          }

          // 创建导航指引UI
          function createNavigationUI(steps) {
            // 处理导航步骤
            navigationSteps = steps.map((step, index) => {
              // 解析指令文本
              const instruction = step.instruction;
              
              // 确定方向图标
              let icon = '↑'; // 默认向前
              
              if (instruction.includes('左转')) {
                icon = '←';
              } else if (instruction.includes('右转')) {
                icon = '→';
              } else if (instruction.includes('掉头')) {
                icon = '↓';
              } else if (instruction.includes('到达终点')) {
                icon = '⚑';
              }
              
              return {
                icon: icon,
                instruction: instruction,
                distance: (step.distance > 1000) ? 
                  (step.distance / 1000).toFixed(1) + '公里' : 
                  Math.round(step.distance) + '米',
                position: step.path[0]
              };
            });
            
            // 设置初始导航指引
            if (navigationSteps.length > 0) {
              currentDirection.value = navigationSteps[0];
              currentStepIndex = 0;
            }
            
            // 激活导航模式
            isNavigating.value = true;
          }

          // 开始实时位置追踪
          function startRealTimeNavigation(path) {
            // 清除之前的监听器
            if (navigationWatcher) {
              navigationWatcher.clear();
            }
            
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
            
            // 设置地图视角
            map.setFitView([routePolyline]);
            
            // 开始监听位置变化
            navigationWatcher = geolocation.watchPosition({
              enableHighAccuracy: true,
              timeout: 5000,
              maximumAge: 0,
              convert: true
            });
            
            // 位置变化回调
            geolocation.on('complete', onLocationUpdate);
            
            // 模拟导航进度（实际应用中应该根据真实位置计算）
            simulateNavigation(path);
          }

          // 位置更新处理
          function onLocationUpdate(result) {
            if (!isNavigating.value) return;
            
            const position = [result.position.lng, result.position.lat];
            const heading = result.heading || 0;
            
            // 更新当前位置标记
            if (currentPositionMarker) {
              currentPositionMarker.setPosition(position);
              currentPositionMarker.setAngle(heading);
            }
            
            // 更新地图中心
            map.setCenter(position);
            
            // 计算导航进度和下一步指引
            updateNavigationProgress(position);
          }

          // 更新导航进度
          function updateNavigationProgress(position) {
            if (!currentRoute || !routePolyline || navigationSteps.length === 0) return;
            
            // 计算到终点的距离
            const endPosition = currentRoute.end;
            const distance = AMap.GeometryUtil.distance(position, endPosition);
            
            // 计算总路程
            const totalDistance = parseFloat(currentRoute.actualDistance);
            
            // 计算进度百分比
            const progress = Math.max(0, Math.min(100, 100 - (distance / (totalDistance * 10))));
            navigationProgress.value = progress;
            
            // 检查是否需要更新导航指引
            if (currentStepIndex < navigationSteps.length - 1) {
              const nextStepPosition = navigationSteps[currentStepIndex + 1].position;
              const distanceToNextStep = AMap.GeometryUtil.distance(position, nextStepPosition);
              
              // 如果接近下一个导航点，更新指引
              if (distanceToNextStep < 30) { // 30米阈值
                currentStepIndex++;
                currentDirection.value = navigationSteps[currentStepIndex];
              }
            }
            
            // 检查是否到达终点
            if (distance < 20) { // 20米阈值
              // 导航完成
              isNavigating.value = false;
              if (navigationWatcher) {
                navigationWatcher.clear();
                geolocation.off('complete', onLocationUpdate);
              }
              
              // 显示到达终点提示
              currentDirection.value = {
                icon: '⚑',
                instruction: '已到达目的地',
                distance: '0米'
              };
            }
          }

          // 模拟导航进度（仅用于演示）
          function simulateNavigation(path) {
            let step = 0;
            const totalSteps = path.length;
            let lastPosition = null;
            
            const interval = setInterval(() => {
              if (!isNavigating.value || step >= totalSteps) {
                clearInterval(interval);
                return;
              }
              
              // 模拟位置更新
              const position = path[step];
              
              // 计算方向角度
              let heading = 0;
              if (step > 0) {
                const prev = path[step - 1];
                const current = path[step];
                heading = Math.atan2(current.lng - prev.lng, current.lat - prev.lat) * 180 / Math.PI;
              }
              
              // 防止位置突变
              if (lastPosition) {
                const distance = AMap.GeometryUtil.distance(
                  [lastPosition.lng, lastPosition.lat], 
                  [position.lng, position.lat]
                );
                
                // 如果两点距离过大（超过50米），则使用插值平滑过渡
                if (distance > 50) {
                  const interpolatedPosition = {
                    lng: lastPosition.lng + (position.lng - lastPosition.lng) * 0.1,
                    lat: lastPosition.lat + (position.lat - lastPosition.lat) * 0.1
                  };
                  
                  // 更新当前位置标记
                  if (currentPositionMarker) {
                    currentPositionMarker.setPosition([interpolatedPosition.lng, interpolatedPosition.lat]);
                    currentPositionMarker.setAngle(heading);
                  }
                  
                  // 更新地图中心
                  map.setCenter([interpolatedPosition.lng, interpolatedPosition.lat]);
                  
                  // 更新导航进度
                  updateNavigationProgress([interpolatedPosition.lng, interpolatedPosition.lat]);
                  
                  lastPosition = interpolatedPosition;
                } else {
                  // 正常更新位置
                  if (currentPositionMarker) {
                    currentPositionMarker.setPosition([position.lng, position.lat]);
                    currentPositionMarker.setAngle(heading);
                  }
                  
                  // 更新地图中心
                  map.setCenter([position.lng, position.lat]);
                  
                  // 更新导航进度
                  updateNavigationProgress([position.lng, position.lat]);
                  
                  lastPosition = position;
                }
              } else {
                // 首次更新位置
                if (currentPositionMarker) {
                  currentPositionMarker.setPosition([position.lng, position.lat]);
                  currentPositionMarker.setAngle(heading);
                }
                
                // 更新地图中心
                map.setCenter([position.lng, position.lat]);
                
                // 更新导航进度
                updateNavigationProgress([position.lng, position.lat]);
                
                lastPosition = position;
              }
              
              step++;
              
              // 模拟进度
              navigationProgress.value = (step / totalSteps) * 100;
              
              // 更新导航指引
              if (step % Math.floor(Math.max(1, totalSteps / navigationSteps.length)) === 0) {
                const stepIndex = Math.min(
                  Math.floor(step / Math.max(1, (totalSteps / navigationSteps.length))),
                  navigationSteps.length - 1
                );
                currentDirection.value = navigationSteps[stepIndex];
              }
              
            }, 800); // 增加更新间隔到800毫秒，使移动更平滑
          }

          // 监听props变化，触发导航
          watch(() => props.startNavigation, (newVal) => {
            if (newVal) {
              startNavigation();
            }
          });

          // 监听路线索引变化
          watch(() => props.currentRouteIndex, (newVal, oldVal) => {
            if (props.routes && props.routes.length > 0 && newVal !== oldVal) {
              console.log('路线索引变化:', newVal);
              // 使用预定义路线数据
              const routeData = props.routes[newVal];
              if (routeData) {
                // 如果有routeAngle属性，使用它来生成固定方向的路线
                if (routeData.routeAngle !== undefined) {
                  const startPosition = currentRoute ? currentRoute.start : [116.397428, 39.90923];
                  const angleInRadians = (routeData.routeAngle * Math.PI) / 180;
                  const distance = routeData.actualDistance ? routeData.actualDistance * 0.009 : 0.009; // 转换为经纬度差
                  const endPosition = [
                    startPosition[0] + distance * Math.cos(angleInRadians),
                    startPosition[1] + distance * Math.sin(angleInRadians)
                  ];
                  
                  // 创建固定路线
                  const fixedRoute = {
                    name: routeData.title,
                    start: startPosition,
                    end: endPosition,
                    distance: routeData.distance,
                    actualDistance: routeData.actualDistance || "1公里",
                    duration: routeData.duration
                  };
                  
                  currentRoute = fixedRoute;
                  
                  // 清除之前的路线
                  if (routePolyline) map.remove(routePolyline);
                  
                  // 使用步行导航生成完整路线
                  walking.search(fixedRoute.start, fixedRoute.end, (status, result) => {
                    if (status === 'complete') {
                      const path = result.routes[0].path;
                      
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
                      map.setFitView([routePolyline]);
                      
                      emit('route-generated', fixedRoute);
                    } else {
                      console.log('步行导航失败，使用备用方案');
                      drawFallbackRoute(fixedRoute);
                    }
                  });
                }
              }
            }
          }, { immediate: true });
          
          // 监听重新生成路线
          watch(() => props.regenerateRoute, (newVal) => {
            if (newVal) {
              if (props.routes && props.routes.length > 0) {
                // 触发currentRouteIndex的watch
                const currentIndex = props.currentRouteIndex;
                const routeData = props.routes[currentIndex];
                if (routeData && routeData.routeAngle !== undefined) {
                  const startPosition = currentRoute ? currentRoute.start : [116.397428, 39.90923];
                  const angleInRadians = (routeData.routeAngle * Math.PI) / 180;
                  const distance = routeData.actualDistance ? routeData.actualDistance * 0.009 : 0.009;
                  const endPosition = [
                    startPosition[0] + distance * Math.cos(angleInRadians),
                    startPosition[1] + distance * Math.sin(angleInRadians)
                  ];
                  
                  const fixedRoute = {
                    name: routeData.title,
                    start: startPosition,
                    end: endPosition,
                    distance: routeData.distance,
                    actualDistance: routeData.actualDistance || "1公里",
                    duration: routeData.duration
                  };
                  
                  currentRoute = fixedRoute;
                  
                  if (routePolyline) map.remove(routePolyline);
                  
                  walking.search(fixedRoute.start, fixedRoute.end, (status, result) => {
                    if (status === 'complete') {
                      const path = result.routes[0].path;
                      
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
                      map.setFitView([routePolyline]);
                      
                      emit('route-generated', fixedRoute);
                    } else {
                      drawFallbackRoute(fixedRoute);
                    }
                  });
                } else {
                  // 如果没有预定义路线，则生成随机路线
                  generateWalkingRoute(currentRoute ? currentRoute.start : [116.397428, 39.90923]);
                }
              } else {
                // 如果没有路线数据，则生成随机路线
                generateWalkingRoute(currentRoute ? currentRoute.start : [116.397428, 39.90923]);
              }
            }
          });
        })
        .catch((e) => {
          console.error('地图加载失败：', e);
        });
    });

    return {
      isNavigating,
      navigationProgress,
      currentDirection
    };
  }
};
</script>

<style scoped>
#container {
  padding: 0px;
  margin: 0px;
  width: 100%;
  height: 100%;
  min-height: 400px;
}

/* 导航指引UI样式 */
.navigation-guide {
  display: none;
  position: absolute;
  bottom: 20px;
  left: 0;
  right: 0;
  margin: 0 auto;
  width: 90%;
  max-width: 400px;
  background-color: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 15px;
  z-index: 1000;
}

.guide-content {
  display: flex;
  align-items: center;
}

.direction-icon {
  font-size: 24px;
  margin-right: 15px;
  width: 40px;
  height: 40px;
  background-color: #3366FF;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.guide-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.guide-instruction {
  display: block;
  font-size: 16px;
  font-weight: 500;
  margin-bottom: 5px;
}

.guide-distance {
  display: block;
  font-size: 14px;
  color: #666;
}

.progress-bar {
  margin-top: 12px;
  height: 4px;
  background-color: #eee;
  border-radius: 2px;
  overflow: hidden;
}

.progress {
  height: 100%;
  background-color: #3366FF;
  transition: width 0.3s ease;
}

/* 强制隐藏高德地图商标和版权信息 */
:deep(.amap-logo),
:deep(.amap-copyright) {
  display: none !important;
  opacity: 0 !important;
  visibility: hidden !important;
}
</style>
