```js
export default {
	data() {
		return {
			navHeight: 95, // 默认高度 状态栏高度 + 内容高度
			statusBarHeight: 40, // 状态栏高度
			contentHeight: 45, // 内容高度
			isNearTop: false, //是否滑动离开顶部
			windowWidth: 375, //可使用窗口宽度
			windowHeight: 730,//可使用窗口高度
			screenHeight: 780,//屏幕高度
			scrollTop: 0, //页面在垂直方向已滚动的距离（单位px）
            lineHeight: 45,//行高
            btnHeight: 45,//胶囊高度
		}
	},
	mounted() {
    this.setSystemInfo() 
	},

	methods: {
    setSystemInfo() {
      // 公式：导航栏高度 = 状态栏到胶囊的间距（胶囊距上边界距离-状态栏高度） * 2 + 胶囊高度 + 状态栏高度。
      uni.getSystemInfo({
      	success: res => {
      		console.log("getSystemInfo：", res)
          const btnInfo = wx.getMenuButtonBoundingClientRect()
          let statusBarHeight = res.statusBarHeight, //状态栏高度
              btnHeight = btnInfo.height, //胶囊高度
              gapHeight = (btnInfo.top - statusBarHeight) * 2,//状态栏到胶囊的间距+胶囊到内容的间距
              navHeight = gapHeight + btnHeight + statusBarHeight  //导航栏高度
          this.btnHeight = btnHeight
          this.navHeight = navHeight
          this.lineHeight = btnInfo.height    
      	 this.statusBarHeight = res.statusBarHeight //  + 7 // 胶囊的上边距7px
      		// this.contentHeight = /iOS/ig.test(res.system) ? 48 : 45 // ios胶囊区域48px，安卓胶囊区域40px
      		// this.navHeight = this.statusBarHeight + this.contentHeight
      		this.windowHeight = res.windowHeight
      		this.screenHeight = res.screenHeight
      	}
      });
      
    }
	},
	onPageScroll(e) {
		this.scrollTop = e.scrollTop //scrollTop页面在垂直方向已滚动的距离（单位px）
		this.isNearTop = this.scrollTop > this.navHeight
	}
}
```

