<script>
	import { HTTP_REQUEST_URL } from './config/app';
	import Auth from './libs/wechat.js';
	import Routine from './libs/routine.js';
	export default {
		globalData: {
			spid: 0,
			code: 0,
			isLogin: false,
			userInfo: {},
			MyMenus: [],
			windowHeight: 0,
			id: 0
		},
		onLaunch: function(option) {
			let that = this;
      // 获得全局的 window 高度
			// #ifdef H5
			uni.getSystemInfo({
				success: function(res) {
					// 首页没有title获取的整个页面的高度，里面的页面有原生标题要减掉就是视口的高度
					// 状态栏是动态的可以拿到 标题栏是固定写死的是44px
					let height = res.windowHeight - res.statusBarHeight - 44
					// #ifdef H5
					that.globalData.windowHeight = res.windowHeight + 'px'
					// #endif
				}
			});
			// #endif

      // 获取导航高度；
      uni.getSystemInfo({
        success: function(res) {
          that.globalData.navHeight = res.statusBarHeight * (750 / res.windowWidth) + 91;
        }
      });
      // #ifdef MP
      let menuButtonInfo = uni.getMenuButtonBoundingClientRect();
      that.globalData.navH = menuButtonInfo.top * 2 + menuButtonInfo.height / 2;
      // #endif

			// #ifdef MP
			if (HTTP_REQUEST_URL === '') {
				console.error(
					"请配置根目录下的config.js文件中的 'HTTP_REQUEST_URL'\n\n请修改开发者工具中【详情】->【AppID】改为自己的Appid\n\n请前往后台【小程序】->【小程序配置】填写自己的 appId and AppSecret"
				);
				return false;
			}
      // TODO 芋艿: 分销
			if (option.query.hasOwnProperty('scene')) {
				switch(option.scene){
					case 1047: // 扫描小程序码
					case 1048: // 长按图片识别小程序码
					case 1049: // 手机相册选取小程序码
					case 1001: // 直接进入小程序
					let value = this.$util.getUrlParams(decodeURIComponent(option.query.scene));
					let values = value.split(',');
					if(values.length === 2){
						let v1 = values[0].split(":");
						if (v1[0] === 'pid') {
							that.globalData.spid = v1[1];
						} else{
							that.globalData.id = v1[1];
						}
						let v2 = values[1].split(":");
						if (v2[0] === 'pid') {
							that.globalData.spid = v2[1];
						}else{
							that.globalData.id = v2[1];
						}
					}else{
						that.globalData.spid = values[0].split(":")[1];
					}
					break;
				}
			}
			// #endif

			// #ifdef H5
      // 静默登录的情况一：微信公众号
			let snsapiBase = 'snsapi_base';
			let urlData = location.pathname + location.search;
			if (!that.$store.getters.isLogin && Auth.isWeixin()) {
				const { code, state } = option.query;
				if (code && code !== uni.getStorageSync('snsapiCode')
          && location.pathname.indexOf('/pages/users/wechat_login/index') === -1) {
          // 存储静默授权code
					uni.setStorageSync('snsapiCode', code);
					Auth.auth(code, state, that.$Cache.get('spread')).then(res => {
            // TODO 芋艿：snRouter 的作用是啥
            uni.setStorageSync('snRouter', decodeURIComponent(decodeURIComponent(option.query.back_url)));
            // 跳转回去
            location.replace(decodeURIComponent(decodeURIComponent(option.query.back_url)));
          }).catch(error => {
            this.$Cache.set('snsapiKey', code);
            // TODO 芋艿：为什么没有 snsapiKey 就反复认证？？？没看懂
            // if (!this.$Cache.has('snsapiKey')) {
            // 	if (location.pathname.indexOf('/pages/users/wechat_login/index') === -1) {
            // 		Auth.oAuth(snsapiBase, option.query.back_url);
            // 	}
            // }
          });
				} else {
					if (!this.$Cache.has('snsapiKey')) {
						if (location.pathname.indexOf('/pages/users/wechat_login/index') === -1) {
							Auth.oAuth(snsapiBase, urlData);
						}
					}
				}
			} else {
				if (option.query.back_url) {
					location.replace(uni.getStorageSync('snRouter'));
				}
			}
			// #endif

			// #ifdef MP
			// 静默登录的情况二：微信小程序
			if (!this.$store.getters.isLogin) {
				let spread = that.globalData.spid ? that.globalData.spid : 0;
				Routine.getCode().then(code => {
          Routine.authUserInfo(code, spread)
            .then(res => {})
        }).catch(res => {
          uni.hideLoading();
        });
			}
			// #endif
		},
		async mounted() {
      // 读取用户的基本信息
      if (this.$store.getters.isLogin && !this.$Cache.get('USER_INFO')) {
        await this.$store.dispatch('USERINFO');
      }
		},
		onShow: function() {
			// #ifdef H5
			uni.getSystemInfo({
				success(e) {
          // TODO 芋艿：这样是否合理？？？
					/* 窗口宽度大于420px且不在PC页面且不在移动设备时跳转至 PC.html 页面 */
					if (e.windowWidth > 420 && !window.top.isPC && !/iOS|Android/i.test(e.system)) {
						// window.location.pathname = 'https://java.crmeb.net/';
						/* 若你的项目未设置根目录（默认为 / 时），则使用下方代码 */
						window.location.pathname = '/static/html/pc.html';
					}
				}
			})
			// #endif
		}
	}
</script>
<style>
	@import url("@/plugin/animate/animate.min.css");
	@import 'static/css/base.css';
	@import 'static/iconfont/iconfont.css';
	@import 'static/css/guildford.css';
	@import 'static/css/style.scss';

	/* 条件编译，仅在H5平台生效 */
	// #ifdef H5
	body::-webkit-scrollbar,
	html::-webkit-scrollbar {
		display: none;
	}

	// #endif
	view {
		box-sizing: border-box;
	}

	.bg-color-red {
		background-color: #E93323 !important;
	}

	.syspadding {
		padding-top: var(--status-bar-height);
	}

	.flex {
		display: flex;
	}

	.uni-scroll-view::-webkit-scrollbar {
		/* 隐藏滚动条，但依旧具备可以滚动的功能 */
		display: none
	}

	::-webkit-scrollbar {
		width: 0;
		height: 0;
		color: transparent;
	}
</style>
