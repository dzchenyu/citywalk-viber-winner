<template>
  <view class="container">
    <!-- 头部导航 -->
    <view class="header">
      <view class="logo-section">
        <i class="fas fa-walking text-xl" style="color: rgba(253, 221, 220, 1)"></i>
        <text class="logo-text">CityWalk</text>
      </view>
      <view class="header-actions">
        <i class="fas fa-search" style="color: rgba(255, 255, 255, 0.8)"></i>
        <i class="fas fa-navicon" style="color: rgba(255, 255, 255, 0.8)"></i>
      </view>
    </view>

    <!-- 时间选择器 -->
    <view class="time-selector">
      <view class="time-options">
        <button 
          class="time-option active" 
          @tap="selectTime('30min')"
        >
          <view class="time-circle">
            <text class="time-text">0.5</text>
            <text class="time-unit">小时</text>
          </view>
          <text class="time-label">散步</text>
        </button>
        
        <button 
          class="time-option" 
          @tap="selectTime('2h')"
        >
          <view class="time-circle inactive">
            <text class="time-text">2</text>
            <text class="time-unit">小时</text>
          </view>
          <text class="time-label">闲逛</text>
        </button>
        
        <button 
          class="time-option" 
          @tap="selectTime('halfday')"
        >
          <view class="time-circle inactive">
            <text class="time-text">半天</text>
          </view>
          <text class="time-label">郊游</text>
        </button>
      </view>
    </view>

    <!-- 地图区域 -->
    <view class="map-container">
      <MapContainer />
      
      <!-- 路线卡片 -->
      <scroll-view class="route-cards" scroll-x="true">
        <view class="route-card glass-container">
          <text class="route-title">文艺咖啡小径</text>
          <view class="route-info">
            <i class="fas fa-route mr-1"></i>
            <text>0.8公里 · 约15分钟</text>
          </view>
          <view class="route-images">
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/c7a85cbb-2a0a-45bf-b48a-bc8cb0aa95ce.jpeg"
              mode="aspectFill"
            ></image>
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/66b3dd15-e32c-42da-b7f8-c4400c5d7a1c.jpeg"
              mode="aspectFill"
              src="https://static.paraflowcontent.com/public/resource/image/3334c00d-1663-4a07-ae67-da633a6e6192.jpeg"
              mode="aspectFill"
            ></image>
          </view>
        </view>
        
        <view class="route-point" style="top: 220rpx; left: 480rpx;">
          <view class="route-point-container">
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/0e5a8d06-759f-4e73-8e35-fc37719b0a4a.jpeg"
              mode="aspectFill"
            ></image>
          </view>
        </view>
        
        <view class="route-point" style="top: 400rpx; left: 320rpx;">
          <view class="route-point-container">
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/3e0c7b8a-d118-4dae-9bef-b3d32f6a2456.jpeg"
              mode="aspectFill"
            ></image>
          </view>
        </view>
      </view>
      
      <!-- 路线卡片 -->
      <scroll-view class="route-cards" scroll-x="true">
        <view class="route-card glass-container">
          <text class="route-title">文艺咖啡小径</text>
          <view class="route-info">
            <i class="fas fa-route mr-1"></i>
            <text>0.8公里 · 约15分钟</text>
          </view>
          <view class="route-images">
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/c7a85cbb-2a0a-45bf-b48a-bc8cb0aa95ce.jpeg"
              mode="aspectFill"
            ></image>
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/66b3dd15-e32c-42da-b7f8-c4400c5d7a1c.jpeg"
              mode="aspectFill"
            ></image>
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/c019f221-8d9a-4146-b239-f3dc6a2fc9ef.jpeg"
              mode="aspectFill"
            ></image>
          </view>
          <view class="route-badge">
            <text>30</text>
          </view>
        </view>
        
        <view class="route-card spot-container">
          <text class="route-title">历史建筑之旅</text>
          <view class="route-info">
            <i class="fas fa-route mr-1"></i>
            <text>1.2公里 · 约20分钟</text>
          </view>
          <view class="route-images">
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/4a0af5c7-7923-47a8-a87b-5ddab37d1fa4.jpeg"
              mode="aspectFill"
            ></image>
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/570a62cd-d4d3-497e-ba1a-14e6ac7ce10e.jpeg"
              mode="aspectFill"
            ></image>
            <image 
              src="https://static.paraflowcontent.com/public/resource/image/2199574c-e00b-4258-a56c-2fad74392e1f.jpeg"
              mode="aspectFill"
            ></image>
          </view>
          <view class="route-badge">
            <text>2h</text>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 底部导航 -->
    <view class="bottom-nav">
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
export default {
  data() {
    return {
      route1Images: [
        'https://static.paraflowcontent.com/public/resource/image/c7a85cbb-2a0a-45bf-b48a-bc8cb0aa95ce.jpeg',
        'https://static.paraflowcontent.com/public/resource/image/66b3dd15-e32c-42da-b7f8-c4400c5d7a1c.jpeg',
        'https://static.paraflowcontent.com/public/resource/image/c019f221-8d9a-4146-b239-f3dc6a2fc9ef.jpeg'
      ],
      route2Images: [
        'https://static.paraflowcontent.com/public/resource/image/4a0af5c7-7923-47a8-a87b-5ddab37d1fa4.jpeg',
        'https://static.paraflowcontent.com/public/resource/image/570a62cd-d4d3-497e-ba1a-14e6ac7ce10e.jpeg',
        'https://static.paraflowcontent.com/public/resource/image/2199574c-e00b-4258-a56c-2fad74392e1f.jpeg'
      ]
    }
  },
  methods: {
    selectTime(time) {
      console.log('选择时间:', time)
    },
    startNavigation() {
      console.log('开始导航')
    },
    dislike() {
      console.log('不喜欢')
    },
    bookmark() {
      console.log('收藏')
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

// 头部导航
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
      font-weight: 300;
    }
  }
  
  .header-actions {
    display: flex;
    gap: 64rpx;
    font-size: 36rpx;
  }
}

// 时间选择器
.time-selector {
  display: flex;
  justify-content: center;
  margin-bottom: 32rpx;
  
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
  }
  
  .time-option {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: none;
    border: none;
    padding: 0;
    
    .time-circle {
      width: 64rpx;
      height: 64rpx;
      background: rgba(253, 221, 220, 0.8);
      border-radius: 50%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      margin-bottom: 8rpx;
      box-shadow: 6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                  -6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                  0 -10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset,
                  0 10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset;
      
      &.inactive {
        background: rgba(41, 36, 33, 0.1);
        box-shadow: 1rpx 2rpx 24rpx 0 rgba(41, 36, 33, 0.08) inset;
      }
    }
    
    .time-text {
      font-size: 20rpx;
      font-weight: 600;
      color: #370b0b;
      line-height: 1;
    }
    
    .time-unit {
      font-size: 12rpx;
      font-weight: 500;
      color: #370b0b;
      line-height: 1;
      margin-top: 2rpx;
    }
    
    .time-label {
      font-size: 24rpx;
      color: #ffffff;
      font-weight: 300;
    }
  }
}

// 地图区域
.map-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  
  .map-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
  
  .location-marker {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    
    .location-marker-inner {
      width: 64rpx;
      height: 64rpx;
      background: rgba(253, 221, 220, 0.8);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      box-shadow: 6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                  -6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                  0 -10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset,
                  0 10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset;
    }
  }
  
  .route-points {
    .route-point {
      position: absolute;
      
      .route-point-container {
        width: 80rpx;
        height: 80rpx;
        border-radius: 50%;
        overflow: hidden;
        border: 4rpx solid #ffffff;
        
        image {
          width: 100%;
          height: 100%;
        }
      }
    }
  }
  
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
      background: rgba(255, 255, 255, 0.05);
      backdrop-filter: blur(6px);
      border-radius: 44rpx;
      position: relative;
      
      &.glass-container {
        background: rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(6px);
      }
      
      &.spot-container {
        background: rgba(41, 36, 33, 0.1);
      }
      
      .route-title {
        font-size: 32rpx;
        color: #ffffff;
        font-weight: 300;
      }
      
      .route-info {
        display: flex;
        align-items: center;
        margin-top: 16rpx;
        opacity: 0.8;
        font-size: 28rpx;
        color: #ffffff;
        
        .fas {
          margin-right: 8rpx;
        }
      }
      
      .route-images {
        display: flex;
        gap: 16rpx;
        margin-top: 24rpx;
        
        image {
          width: 120rpx;
          height: 120rpx;
          border-radius: 16rpx;
        }
      }
      
      .route-badge {
        position: absolute;
        top: 16rpx;
        right: 16rpx;
        width: 48rpx;
        height: 48rpx;
        background: rgba(253, 221, 220, 0.8);
        border-radius: 50%;
        display: flex;
        justify-content: center;
        align-items: center;
        box-shadow: 6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                    -6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                    0 -10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset,
                    0 10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset;
        
        text {
          font-size: 24rpx;
          color: #370b0b;
          font-weight: 600;
        }
        
        &.spot-container {
          background: rgba(41, 36, 33, 0.1);
          
          text {
            color: #ffffff;
          }
        }
      }
    }
  }
}

// 底部导航
.bottom-nav {
  margin-top: 48rpx;
  
  .nav-container {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin: 0 48rpx 24rpx;
    padding: 32rpx 16rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(6px);
    border-radius: 88rpx;
  }
  
  .nav-button {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: none;
    border: none;
    padding: 0;
    flex: 1;
    
    .nav-circle {
      width: 96rpx;
      height: 96rpx;
      background: rgba(41, 36, 33, 0.1);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 16rpx;
      
      &.primary {
        background: rgba(253, 221, 220, 0.8);
        box-shadow: 6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                    -6rpx 0 8rpx -8rpx rgba(255, 255, 255, 1) inset,
                    0 -10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset,
                    0 10rpx 8rpx -8rpx rgba(188, 139, 134, 0.4) inset;
      }
    }
    
    .nav-text {
      font-size: 24rpx;
      color: rgba(255, 255, 255, 0.5);
      
      &.primary {
        color: #ffffff;
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
}

// 通用样式
.strong-glass {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(6px);
}

.glass-container {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(6px);
}

.spot-container {
  background: rgba(41, 36, 33, 0.1);
}
</style>
