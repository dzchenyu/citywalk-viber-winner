<template>
  <view class="container">
    <!-- 头部导航栏：包含logo和搜索/菜单按钮 -->
    <view class="header">
      <view class="logo-section">
        <!-- 步行图标和CityWalk品牌名称 -->
        <i class="fas fa-walking fa-2x" style="color: rgba(253, 221, 220, 1)"></i>
        <text class="logo-text" >第二条街</text>
      </view>
      <view class="header-actions">
        <!-- 搜索和菜单图标 -->
        <i class="fas fa-search" style="color: rgba(255, 255, 255, 0.8)"></i>
        <i class="fas fa-navicon" style="color: rgba(255, 255, 255, 0.8)"></i>
      </view>
    </view>

    <!-- 时间选择器：用户可以选择散步时长 -->
    <view class="time-selector">
      <view class="time-options">
        <!-- 30分钟选项：默认选中状态 -->
        <button 
          :class="['time-option', selectedTime === '30min' ? 'active' : '']"
          @tap="selectTime('30min')"
        >
          <!-- 时间圆圈：未选中时显示inactive样式 -->
          <view :class="['time-circle', selectedTime !== '30min' ? 'inactive' : '']">
            <text class="time-text">0.5</text>
            <text class="time-unit">小时</text>
          </view>
          <text class="time-label">散步</text>
        </button>
        
        <!-- 2小时选项 -->
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
        
        <!-- 半天选项 -->
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

    <!-- 地图区域：显示高德地图和路线卡片 -->
    <view class="map-container">
      <!-- 高德地图组件 -->
      <MapContainer 
        :regenerate-route="regenerateRoute"
        @route-generated="onRouteGenerated"
        @navigation-ready="onNavigationReady"
      />
      
      <!-- 路线卡片：横向滚动的路线推荐 -->
      <scroll-view class="route-cards" scroll-x="true">
        <!-- 动态渲染当前时间对应的路线卡片 -->
        <view 
          v-for="(route, index) in currentRoutes" 
          :key="index"
          :class="['route-card', route.type]"
        >
          <text class="route-title">{{ route.title }}</text>
          <view class="route-info">
            <i class="fas fa-route mr-1"></i>
            <text>{{ route.distance }} · {{ route.duration }}</text>
          </view>
          <view class="route-images">
            <!-- 路线相关的图片展示 -->
            <image 
              v-for="(img, imgIndex) in route.images" 
              :key="imgIndex"
              :src="img"
              mode="aspectFill"
            ></image>
          </view>
          <!-- 时间标签：显示当前选择的时间 -->
          <view class="route-badge">
            <text>{{ currentTimeLabel }}</text>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 底部导航：三个操作按钮 -->
    <view class="bottom-nav">
      <view class="nav-container">
        <!-- 不喜欢按钮 -->
        <button class="nav-button" @tap="dislike">
          <view class="nav-circle">
            <i class="fas fa-thumbs-down text-lg" style="color: rgba(255, 255, 255, 0.5)"></i>
          </view>
          <text class="nav-text">不喜欢</text>
        </button>
        
        <!-- 开始导航按钮：主要操作 -->
        <button class="nav-button" @tap="startNavigation">
          <view class="nav-circle primary">
            <i class="fas fa-location-arrow text-lg" style="color: rgba(55, 11, 11, 1)"></i>
          </view>
          <text class="nav-text primary">开始导航</text>
        </button>
        
        <!-- 收藏按钮 -->
        <button class="nav-button" @tap="bookmark">
          <view class="nav-circle">
            <i class="fas fa-bookmark text-lg" style="color: rgba(255, 255, 255, 0.5)"></i>
          </view>
          <text class="nav-text">收藏</text>
        </button>
      </view>
      
      <!-- 底部指示器 -->
      <view class="home-indicator">
        <view class="indicator-bar"></view>
      </view>
    </view>
  </view>
</template>

<script>
// 导入地图组件
import MapContainer from '../MapContainer.vue'

export default {
  components: {
    MapContainer
  },
  data() {
    return {
      // 当前选中的时间选项，默认为30分钟
      selectedTime: '30min',
      // 当前激活的底部导航按钮
      activeNav: null,
      // 导航触发标志，用于通知地图组件开始导航
      startNavigationFlag: false,
      // 当前生成的路线信息
      currentRoute: null,
      // 重新生成路线标志
      regenerateRoute: false,
      // 当前卡片索引
      currentCardIndex: 0,
      // 卡片切换动画状态
      cardTransition: false,
      // 路线数据，按时间分类
      routesData: {
        '30min': [
          {
            title: '文艺咖啡小径',
            distance: '0.8公里',
            duration: '约15分钟',
            images: [
              'https://static.paraflowcontent.com/public/resource/image/c7a85cbb-2a0a-45bf-b48a-bc8cb0aa95ce.jpeg',
              'https://static.paraflowcontent.com/public/resource/image/66b3dd15-e32c-42da-b7f8-c4400c5d7a1c.jpeg',
              'https://www.bizhigq.com/pc-img/2023-06/g3372.jpg'
            ],
            type: 'enhanced-card'
          },
          {
            title: '历史建筑之旅',
            distance: '1.2公里',
            duration: '约20分钟',
            images: [
              'https://static.paraflowcontent.com/public/resource/image/4a0af5c7-7923-47a8-a87b-5ddab37d1fa4.jpeg',
              'https://static.paraflowcontent.com/public/resource/image/570a62cd-d4d3-497e-ba1a-14e6ac7ce10e.jpeg',
              'https://static.paraflowcontent.com/public/resource/image/2199574c-e00b-4258-a56c-2fad74392e1f.jpeg'
            ],
            type: 'dark-card'
          }
        ],
        '2h': [
          {
            title: '城市公园漫步',
            distance: '3.5公里',
            duration: '约45分钟',
            images: [
              'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=400',
              'https://images.unsplash.com/photo-1447752875215-b2761acb3c5d?w=400',
              'https://images.unsplash.com/photo-1469474968028-56623f02e42e?w=400'
            ],
            type: 'enhanced-card'
          },
          {
            title: '艺术街区探索',
            distance: '2.8公里',
            duration: '约35分钟',
            images: [
              'https://images.unsplash.com/photo-1511406361295-0a1ff814c0ce?w=400',
              'https://images.unsplash.com/photo-1541961017774-22349e4a1262?w=400',
              'https://images.unsplash.com/photo-1578662996442-48f60103fc96?w=400'
            ],
            type: 'dark-card'
          }
        ],
        'halfday': [
          {
            title: '郊外山野徒步',
            distance: '8.2公里',
            duration: '约2.5小时',
            images: [
              'https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?w=400',
              'https://images.unsplash.com/photo-1501785888041-af3ef285b470?w=400',
              'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=400'
            ],
            type: 'enhanced-card'
          },
          {
            title: '古镇文化之旅',
            distance: '6.5公里',
            duration: '约2小时',
            images: [
              'https://images.unsplash.com/photo-1519501025264-65ba15a82390?w=400',
              'https://images.unsplash.com/photo-1465447142348-e9952c393450?w=400',
              'https://images.unsplash.com/photo-1477959858617-67f85cf4f1df?w=400'
            ],
            type: 'dark-card'
          }
        ]
      }
    }
  },
  computed: {
    // 根据当前选择的时间获取对应的路线
    currentRoutes() {
      return this.routesData[this.selectedTime] || []
    },
    // 获取当前时间标签
    currentTimeLabel() {
      const labels = {
        '30min': '30',
        '2h': '2h',
        'halfday': '半天'
      }
      return labels[this.selectedTime] || '30'
    }
  },
  methods: {
    // 选择时间：更新选中状态并记录日志，同时重新生成路线
    selectTime(time) {
      this.selectedTime = time
      console.log('选择时间:', time)
      
      // 触发地图重新生成路线
      this.regenerateRoute = true
      setTimeout(() => {
        this.regenerateRoute = false
      }, 100)
    },
    // 开始导航：触发地图组件的导航功能
    startNavigation() {
      this.activeNav = 'navigate'
      this.startNavigationFlag = true
      console.log('开始导航到:', this.currentRoute?.name || '当前路线')
      
      // 2秒后重置导航标志
      setTimeout(() => {
        this.startNavigationFlag = false
      }, 2000)
    },
    // 不喜欢：设置激活状态并记录日志
    dislike() {
      this.activeNav = 'dislike'
      console.log('不喜欢')
    },
    // 收藏：设置激活状态并记录日志
    bookmark() {
      this.activeNav = 'bookmark'
      console.log('收藏')
    },
    // 路线生成回调：接收地图组件生成的路线信息
    onRouteGenerated(route) {
      this.currentRoute = route
      console.log('生成路线:', route)
    },
    // 导航准备就绪回调
    onNavigationReady(result) {
      console.log('导航准备就绪:', result)
    }
  }
}
</script>

<style lang="scss" scoped>
// 导入字体图标
@import url('https://static.paraflowcontent.com/public/css/font-awesome/font-awesome.672.css');

// 主容器样式：全屏渐变背景
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

// 头部导航样式
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

// 增强的时间选择器样式
.time-selector {
  display: flex;
  justify-content: center;
  padding:10px;
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
    margin-top:10px;
    padding: 20;
    transition: all 0.3s ease;
    
    &:active {
      transform: scale(0.95);
    }
    
    // 激活状态样式
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
      
      // 未激活状态样式
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

// 地图容器样式
.map-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  height: 600px; // 固定高度确保地图正常显示
}

// 增强的路线卡片样式
.route-cards {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 0 48rpx 32rpx;
  white-space: nowrap;
  
  .route-card {
    display: inline-block;
    width: 560rpx;
    padding: 32rpx;
    margin-right: 32rpx;
    border-radius: 44rpx;
    position: relative;
    border: 2rpx solid rgba(255, 255, 255, 0.2);
    
    // 浅色卡片样式
    &.enhanced-card {
      background: linear-gradient(135deg, 
        rgba(114, 107, 97, 0.9) 0%, 
        rgba(93, 89, 81, 0.9) 100%);
      backdrop-filter: blur(12px);
      box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.3),
                  0 2rpx 8rpx rgba(0, 0, 0, 0.2);
    }
    
    // 深色卡片样式
    &.dark-card {
      background: linear-gradient(135deg, 
        rgba(55, 11, 11, 0.9) 0%, 
        rgba(41, 36, 33, 0.9) 100%);
      backdrop-filter: blur(12px);
      box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.4),
                  0 2rpx 8rpx rgba(0, 0, 0, 0.3);
    }
    
    .route-title {
      font-size: 36rpx;
      color: #ffffff;
      font-weight: 600;
      margin-bottom: 12rpx;
      text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.5);
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
      
      image {
        width: 110rpx;
        height: 110rpx;
        border-radius: 12rpx;
        border: 2rpx solid rgba(255, 255, 255, 0.3);
        box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.2);
      }
    }
    
    .route-badge {
      position: absolute;
      top: 20rpx;
      right: 20rpx;
      width: 56rpx;
      height: 56rpx;
      background: linear-gradient(135deg, 
        rgba(253, 221, 220, 0.9) 0%, 
        rgba(188, 139, 134, 0.9) 100%);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.3);
      
      text {
        font-size: 24rpx;
        color: #370b0b;
        font-weight: 700;
      }
    }
  }
}

// 底部导航增强样式
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
      
      // 主要按钮样式（开始导航）
      &.primary {
        background: linear-gradient(135deg, 
          rgba(253, 221, 220, 0.9) 0%, 
          rgba(188, 139, 134, 0.9) 100%);
        box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.3),
                    0 0 0 2rpx rgba(255, 255, 255, 0.2);
      }
      
      // 激活状态样式
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

// 底部指示器样式
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
</style>
