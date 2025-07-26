<template>
  <view class="container">
    <!-- 头部导航栏：包含logo和搜索/菜单按钮 -->
    <view class="header" v-if="!isFullScreenNavigation">
      <view class="logo-section">
        <i class="fas fa-walking fa-2x" style="color: rgba(253, 221, 220, 1)"></i>
        <text class="logo-text">第二条街</text>
      </view>
      <view class="header-actions">
        <i class="fas fa-search" style="color: rgba(255, 255, 255, 0.8)"></i>
        <i class="fas fa-navicon" style="color: rgba(255, 255, 255, 0.8)"></i>
      </view>
    </view>

    <!-- 时间选择器：用户可以选择散步时长 -->
    <view class="time-selector" v-if="!isFullScreenNavigation">
      <view class="time-options">
        <button 
          :class="['time-option', selectedTime === '30min' ? 'active' : '']"
          @tap="selectTime('30min')"
        >
          <view :class="['time-circle', selectedTime !== '30min' ? 'inactive' : '']">
            <text class="time-text">0.5</text>
            <text class="time-unit">小时</text>
          </view>
          <text class="time-label">散步</text>
        </button>
        
        <button 
          :class="['time-option', selectedTime === '2h' ? 'active' : '']"
          @tap="selectTime('2h')"
        >
          <view :class="['time-circle', selectedTime !== '2h' ? 'inactive' : '']">
            <text class="time-text">2</text>
            <text class="time-unit">小时</text>
          </view>
          <text class="time-label">闲逛</text>
        </button>
        
        <button 
          :class="['time-option', selectedTime === 'halfday' ? 'active' : '']"
          @tap="selectTime('halfday')"
        >
          <view :class="['time-circle', selectedTime !== 'halfday' ? 'inactive' : '']">
            <text class="time-text">半天</text>
          </view>
          <text class="time-label">郊游</text>
        </button>
      </view>
    </view>

    <!-- 地图区域：显示高德地图和单卡片 -->
    <view :class="['map-container', { 'fullscreen': isFullScreenNavigation }]">
      <!-- 高德地图组件 -->
      <MapContainer 
        ref="mapContainer"
        :regenerate-route="regenerateRoute"
        :current-route-index="currentCardIndex"
        :routes="currentRoutes"
        :start-navigation="startNavigationFlag"
        @route-generated="onRouteGenerated"
        @navigation-ready="onNavigationReady"
      />
      
      <!-- 单卡片展示：淡入淡出效果 -->
      <view class="single-card-container" v-if="!isFullScreenNavigation">
        <transition name="fade" mode="out-in">
          <view 
            :class="['single-route-card', currentRoutes[currentCardIndex]?.type || 'enhanced-card']"
            v-if="currentRoutes.length > 0"
            :key="currentCardIndex"
          >
            <view class="card-header">
              <text class="route-title">{{ currentRoutes[currentCardIndex]?.title }}</text>
              <view class="route-badge">
                <text>{{ currentTimeLabel }}</text>
              </view>
            </view>
            
            <view class="route-info">
              <i class="fas fa-route"></i>
              <text>{{ currentRoutes[currentCardIndex]?.distance }} · {{ currentRoutes[currentCardIndex]?.duration }}</text>
            </view>
            
            <view class="route-images">
              <image 
                v-for="(img, imgIndex) in currentRoutes[currentCardIndex]?.images" 
                :key="imgIndex"
                :src="img"
                mode="aspectFill"
                class="route-image"
              ></image>
            </view>
            
            <view class="card-progress">
              <text class="progress-text">{{ currentCardIndex + 1 }} / {{ currentRoutes.length }}</text>
            </view>
          </view>
        </transition>
      </view>
      
      <!-- 全屏导航时的返回按钮 -->
      <view class="navigation-header" v-if="isFullScreenNavigation">
        <view class="back-button" @click="exitNavigation" @tap="exitNavigation">
          <i class="fas fa-arrow-left"></i>
          <text>返回</text>
        </view>
        <view class="navigation-title">
          <text>{{ currentRoutes[currentCardIndex]?.title }}</text>
        </view>
      </view>
    </view>

    <!-- 底部导航：三个操作按钮 -->
    <view class="bottom-nav" v-if="!isFullScreenNavigation">
      <view class="nav-container">
        <button class="nav-button" @tap="dislike">
          <view class="nav-circle">
            <i class="fas fa-thumbs-down text-lg" style="color: rgba(255, 255, 255, 0.5)"></i>
          </view>
          <text class="nav-text">不喜欢</text>
        </button>
        
        <button class="nav-button" @tap="startNavigation">
          <view class="nav-circle primary">
            <i class="fas fa-location-arrow text-lg" style="color: rgba(55, 11, 11, 1)"></i>
          </view>
          <text class="nav-text primary">开始导航</text>
        </button>
        
        <button class="nav-button" @tap="bookmark">
          <view class="nav-circle">
            <i class="fas fa-bookmark text-lg" style="color: rgba(255, 255, 255, 0.5)"></i>
          </view>
          <text class="nav-text">收藏</text>
        </button>
      </view>
      
      <view class="home-indicator">
        <view class="indicator-bar"></view>
      </view>
    </view>
  </view>
</template>

<script>
import MapContainer from '../MapContainer.vue'

export default {
  components: {
    MapContainer
  },
  data() {
    return {
      selectedTime: '30min',
      activeNav: null,
      startNavigationFlag: false,
      currentRoute: null,
      regenerateRoute: false,
      currentCardIndex: 0,
      cardTransition: false,
      isFullScreenNavigation: false,
      routesData: {
        '30min': [
          {
            title: '文艺咖啡小径',
            distance: '0.8公里',
            duration: '约15分钟',
            actualDistance: 0.8,
            images: [
              'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1442512595331-e89e73853f31?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'enhanced-card',
            routeAngle: 45
          },
          {
            title: '历史建筑之旅',
            distance: '1.2公里',
            duration: '约20分钟',
            actualDistance: 1.2,
            images: [
              'https://images.unsplash.com/photo-1519501025264-65ba15a82390?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1465447142348-e9952c393450?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1477959858617-67f85cf4f1df?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'dark-card',
            routeAngle: 135
          }
        ],
        '2h': [
          {
            title: '城市公园漫步',
            distance: '3.5公里',
            duration: '约45分钟',
            actualDistance: 3.5,
            images: [
              'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1447752875215-b2761acb3c5d?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1469474968028-56623f02e42e?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'enhanced-card',
            routeAngle: 60
          },
          {
            title: '艺术街区探索',
            distance: '2.8公里',
            duration: '约35分钟',
            actualDistance: 2.8,
            images: [
              'https://images.unsplash.com/photo-1511406361295-0a1ff814c0ce?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1541961017774-22349e4a1262?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1578662996442-48f60103fc96?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'dark-card',
            routeAngle: 240
          }
        ],
        'halfday': [
          {
            title: '郊外山野徒步',
            distance: '8.2公里',
            duration: '约2.5小时',
            actualDistance: 8.2,
            images: [
              'https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'enhanced-card',
            routeAngle: 90
          },
          {
            title: '古镇文化之旅',
            distance: '6.5公里',
            duration: '约2小时',
            actualDistance: 6.5,
            images: [
              'https://images.unsplash.com/photo-1519501025264-65ba15a82390?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1465447142348-e9952c393450?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80',
              'https://images.unsplash.com/photo-1477959858617-67f85cf4f1df?ixlib=rb-1.2.1&auto=format&fit=crop&w=400&h=300&q=80'
            ],
            type: 'dark-card',
            routeAngle: 270
          }
        ]
      }
    }
  },
  computed: {
    currentRoutes() {
      return this.routesData[this.selectedTime] || []
    },
    currentTimeLabel() {
      const labels = {
        '30min': '0.5h',
        '2h': '2h',
        'halfday': '半天'
      }
      return labels[this.selectedTime] || '30'
    }
  },
  mounted() {
    this.$nextTick(() => {
      setTimeout(() => {
        if (this.currentRoutes.length > 0) {
          this.regenerateRoute = true
          setTimeout(() => {
            this.regenerateRoute = false
          }, 200)
        }
      }, 500)
    })
  },
  methods: {
    selectTime(time) {
      this.selectedTime = time
      this.currentCardIndex = 0
      this.regenerateRoute = true
      setTimeout(() => {
        this.regenerateRoute = false
      }, 100)
    },
    startNavigation() {
      this.activeNav = 'navigate'
      this.startNavigationFlag = true
      this.isFullScreenNavigation = true
      
      setTimeout(() => {
        this.startNavigationFlag = false
      }, 2000)
    },
    exitNavigation() {
      this.isFullScreenNavigation = false
      // 重置导航状态，确保导航指引UI被隐藏
      this.startNavigationFlag = false
      // 通知MapContainer组件停止导航
      this.$nextTick(() => {
        const mapContainer = this.$refs.mapContainer
        if (mapContainer && typeof mapContainer.stopNavigation === 'function') {
          mapContainer.stopNavigation()
        }
      })
    },
    dislike() {
      this.activeNav = 'dislike'
      if (this.currentCardIndex < this.currentRoutes.length - 1) {
        this.currentCardIndex++
      } else {
        this.currentCardIndex = 0
      }
    },
    bookmark() {
      this.activeNav = 'bookmark'
    },
    onRouteGenerated(route) {
      this.currentRoute = route
    },
    onNavigationReady(result) {
      console.log('导航准备就绪:', result)
    }
  }
}
</script>

<style lang="scss" scoped>
@import url('https://static.paraflowcontent.com/public/css/font-awesome/font-awesome.672.css');

.container {
  min-height: 100vh;
  background: radial-gradient(68% 68% at 50% 28%, rgba(114, 107, 97, 1) 0%, rgba(93, 89, 81, 1) 100%);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 750rpx;
  min-height: 1624rpx;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  line-height: 1.4;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 112rpx;
  padding: 0 48rpx;
  color: #ffffff;
  
  .logo-section {
    display: flex;
    align-items: center;
    gap: 16rpx;
    
    .logo-text {
      font-size: 40rpx;
      font-weight: 500;
    }
  }
  
  .header-actions {
    display: flex;
    gap: 64rpx;
    font-size: 36rpx;
  }
}

.time-selector {
  display: flex;
  justify-content: center;
  padding: 10px;
  margin-bottom: 30rpx;
  
  .time-options {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 654rpx;
    height: 120rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(6px);
    border-radius: 44rpx;
    padding: 24rpx 32rpx;
    border: 1rpx solid rgba(255, 255, 255, 0.2);
  }
  
  .time-option {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: none;
    border: none;
    margin-top: 10px;
    padding: 20;
    transition: all 0.3s ease;
    
    &:active {
      transform: scale(0.95);
    }
    
    &.active {
      .time-circle {
        background: linear-gradient(135deg, 
          rgba(253, 221, 220, 1) 0%, 
          rgba(188, 139, 134, 1) 100%);
        box-shadow: 0 0 20rpx rgba(253, 221, 220, 0.6),
                    0 0 0 2rpx rgba(255, 255, 255, 0.3);
      }
      
      .time-label {
        color: rgba(253, 221, 220, 1);
        font-weight: 600;
      }
    }
    
    .time-circle {
      width: 80rpx;
      height: 80rpx;
      background: rgba(253, 221, 220, 0.8);
      border-radius: 50%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      margin-bottom: 8rpx;
      transition: all 0.3s ease;
      
      &.inactive {
        background: rgba(41, 36, 33, 0.2);
        box-shadow: 1rpx 2rpx 24rpx 0 rgba(41, 36, 33, 0.08) inset;
      }
    }
    
    .time-text {
      font-size: 30rpx;
      font-weight: 600;
      color: #370b0b;
      line-height: 1;
    }
    
    .time-unit {
      font-size: 15rpx;
      font-weight: 600;
      color: #370b0b;
      line-height: 1;
      margin-top: 2rpx;
    }
    
    .time-label {
      font-size: 24rpx;
      color: #ffffff;
      font-weight: 300;
      transition: all 0.3s ease;
    }
  }
}

.map-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  height: 600px;
  transition: all 0.3s ease;
}

.map-container.fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1000;
}

.single-card-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) translateX(-30rpx) translateY(80rpx);
  width: 580rpx;
  z-index: 100;
  margin-top: 180rpx;
}

.single-route-card {
  width: 100%;
  padding: 28rpx;
  border-radius: 32rpx;
  position: relative;
  border: 1rpx solid rgba(255, 255, 255, 0.15);
  
  &.enhanced-card {
    background: rgba(30, 30, 30, 0.35);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    box-shadow: 0 12rpx 40rpx rgba(0, 0, 0, 0.25),
                0 4rpx 16rpx rgba(0, 0, 0, 0.15),
                inset 0 1rpx 0 rgba(255, 255, 255, 0.05);
  }
  
  &.dark-card {
    background: rgba(20, 20, 20, 0.4);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    box-shadow: 0 12rpx 40rpx rgba(0, 0, 0, 0.3),
                0 4rpx 16rpx rgba(0, 0, 0, 0.2),
                inset 0 1rpx 0 rgba(255, 255, 255, 0.03);
  }
  
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 16rpx;
  }
  
  .route-title {
    font-size: 36rpx;
    color: #fcfcfc;
    font-weight: 600;
    text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.5);
    flex: 1;
  }
  
  .route-info {
    display: flex;
    align-items: center;
    margin-bottom: 24rpx;
    font-size: 28rpx;
    color: rgba(255, 255, 255, 0.9);
    font-weight: 500;
    
    .fas {
      margin-right: 8rpx;
      color: rgba(253, 221, 220, 0.9);
    }
  }
  
  .route-images {
    display: flex;
    gap: 12rpx;
    margin-bottom: 16rpx;
    
    .route-image {
      width: 110rpx;
      height: 110rpx;
      border-radius: 12rpx;
      border: 2rpx solid rgba(255, 255, 255, 0.3);
      box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.2);
    }
  }
  
  .card-progress {
    text-align: center;
    margin-top: 16rpx;
    
    .progress-text {
      font-size: 24rpx;
      color: rgba(255, 255, 255, 0.7);
      font-weight: 500;
    }
  }
}

.navigation-header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 112rpx;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  padding: 0 48rpx;
  z-index: 1001;
}

.back-button {
  display: flex;
  align-items: center;
  background: none;
  border: none;
  color: white;
  font-size: 28rpx;
  gap: 16rpx;
  position: absolute;
  left: 48rpx;
  z-index:1000;//设置堆叠的顺序
}

.navigation-title {
  position: absolute;
  left: 0;
  right: 0;
  text-align: center;
  color: white;
  font-size: 32rpx;
  font-weight: 600;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

.bottom-nav {
  margin-top: 50rpx;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  
  .nav-container {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin: 0 48rpx 24rpx;
    padding: 32rpx 5rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(6px);
    border-radius: 88rpx;
    border: 1rpx solid rgba(255, 255, 255, 0.2);
  }
  
  .nav-button {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: none;
    border: none;
    padding: 0;
    flex: 1;
    transition: all 0.3s ease;
    
    &:active {
      transform: scale(0.95);
    }
    
    .nav-circle {
      width: 96rpx;
      height: 96rpx;
      background: rgba(41, 36, 33, 0.2);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 16rpx;
      transition: all 0.3s ease;
      border: 2rpx solid rgba(255, 255, 255, 0.1);
      
      &.primary {
        background: linear-gradient(135deg, 
          rgba(253, 221, 220, 0.9) 0%, 
          rgba(188, 139, 134, 0.9) 100%);
        box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.3),
                    0 0 0 2rpx rgba(255, 255, 255, 0.2);
      }
      
      &.active {
        background: rgba(253, 221, 220, 0.9);
        box-shadow: 0 0 20rpx rgba(253, 221, 220, 0.5);
      }
    }
    
    .nav-text {
      font-size: 24rpx;
      color: rgba(255, 255, 255, 0.7);
      font-weight: 500;
      transition: all 0.3s ease;
      
      &.primary {
        color: #370b0b;
        font-weight: 600;
      }
      
      &.active {
        color: rgba(253, 221, 220, 1);
        font-weight: 600;
      }
    }
  }
}

.home-indicator {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 68rpx;
  
  .indicator-bar {
    width: 268rpx;
    height: 10rpx;
    background: rgba(255, 255, 255, 0.6);
    border-radius: 10rpx;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.route-cards {
  display: none;
}
</style>
