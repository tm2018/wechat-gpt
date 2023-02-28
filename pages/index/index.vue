<template>
	<view class="bg">
		<view class="advertising">
			<ad unit-id="adunit-94ed6bcc5bb80d7f"></ad>
		</view>
		<scroll-view scroll-with-animation :scroll-y="isScroll" :scroll-top="scrollTop" style="width: 100%;">
			
			<!-- 用来获取消息体高度 -->
			<view id="okk" scroll-with-animation>
				<!-- 消息 -->
				<view class="flex-column-start" v-for="(x,i) in msgList" :key="i">
					<!-- 用户消息 头像可选加入-->
					<view v-if="x.my" class="userinfo">
						<view class="flex justify-end my-info">
							<view class="usermsg" style="max-width: 500rpx;">
								<text style="word-break: break-all;" selectable='true'>{{x.msg}}</text>
							</view>
							<view class="chat-img" style="margin-left:20rpx ;">
								<image style="height: 100rpx;width: 100rpx;" :src="userAvatar" mode="aspectFit">
								</image>
							</view>
						</view>
					</view>
					<!-- 机器人消息 -->
					<view v-if="!x.my" class="aiinfo">
						<view class="chat-img ">
							<image style="height: 100rpx;width: 100rpx;" src="../../static/openai.png" mode="scaleToFill">
							</image>
						</view>
						<view class="flex" style="max-width: 500rpx;">
							<view class="aimsg" style="border-radius: 35rpx;background-color: #f9f9f9;">
								<text style="word-break: break-all;" @longpress='copyText(x.msg)'>{{x.msg}}</text>
								
							</view>
						</view>
					</view>
				</view>
				<!-- 防止消息底部被遮 -->
				<view style="height: 260rpx;">
				</view>
			</view>
		</scroll-view>
		<!-- 底部导航栏 -->
		<view class="flex-column-center"><view class="inpubut" z-else>
<button data-name="shareBtn" open-type="share"  >分享获取对话次数</button>
<button class="avatar-wrapper" open-type="chooseAvatar" @chooseavatar="onChooseAvatar">选择头像</button></view>
			<button class="btn" style="margin-bottom: 50rpx;width: auto;"
				v-if="!apisucc">{{apibut}}</button>
			<view class="inpubut" v-else>
				<input v-model="msg" class="dh-input" type="text" @confirm="sendMsg" confirm-type="search"
					placeholder-class="my-neirong-sm" placeholder="描述您的问题" @blur="isScroll=true;" @focus="isScroll=false;"/>

				<button @click="sendMsg" :disabled="msgLoad" class="btn">{{isRequesting?'请求中...':sentext}}</button>
				
			</view>
		</view>
		
		<!-- <uni-popup ref="popup" type="center">
			<view class="popcls">
				<view class="uni-textarea" style="width: 90%;margin: 20rpx 20rpx;border: 1px solid #000000;">
					<textarea style="width: 100%; " placeholder-style="color:#F76260" :placeholder="apiadj"
						v-model="api" />
				</view>
				<view style="display: flex;flex-direction: row;">
					<button style="margin: 10rpx;" type="primary" @click="apiset">确认</button>
					<button style="margin: 10rpx;" @click="clopop">取消</button>
				</view>
			</view>
		</uni-popup> -->
		<uni-popup ref="popup" type="left">
			<view class="popcls">
				<button class="session_btn" open-type="contact" bindcontact="handleContact" session-from="sessionFrom">联系客服</button>		
				<button class="avatar-wrapper" open-type="chooseAvatar" @chooseavatar="onChooseAvatar">
			选择头像
		        </button>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				rewardedVideoAd:null,//广告
				num:3,//次数
				scrollTop:9999,
				isScroll:true,//是否可以滑动
				userAvatar: '../../static/user.png',//头像
				apiurl: 'https://api.mmlovecc.cc/api/api.php',
				apisucc: true,
				apibut: 'api检测中,请稍等...',
				// sentext: '发送',
				// apiadj: '在此输入你的APIKEY',
				api: '',
				msgLoad: false,
				anData: {},
				isRequesting:false,
				animationData: {},
				showTow: true,
				msgList: [{
					my: false,
					msg: "你好，我是小毛，有什么想和我说的吗？"
				}],
				msgContent: "",
				msg: ""
			}
		},
		computed:{
			sentext:function(){
				return `发送(${this.num}次)`
			}
		},		onShareAppMessage: function( options ){
			this.num = 5;
			uni.setStorageSync('user-num',this.num);
		},
		onLoad() {
			// 激励广告
			if(wx.createRewardedVideoAd){
			      this.rewardedVideoAd = wx.createRewardedVideoAd({ adUnitId: '' })
			      this.rewardedVideoAd.onLoad(() => {
			        console.log('onLoad event emit')
			      })
			      this.rewardedVideoAd.onError((err) => {
			        console.log('onError event emit', err)
			      })
			      this.rewardedVideoAd.onClose((res) => {
			        console.log('onClose event emit', res)
					if(res.isEnded){
						this.num=5
					}
			      })
			    }
			wx.showShareMenu({
			        withShareTicket:true,
			        //设置下方的Menus菜单，才能够让发送给朋友与分享到朋友圈两个按钮可以点击
			        menus:["shareAppMessage","shareTimeline"]
			    })
		
			const that = this;
			if (uni.getStorageSync('user-avatar') !== undefined){
				this.userAvatar = uni.getStorageSync('user-avatar')
			}
			if (uni.getStorageSync('user-num') === 0 ||uni.getStorageSync('user-num') === 1||uni.getStorageSync('user-num') === 2||uni.getStorageSync('user-num') === 3||uni.getStorageSync('user-num') === 4||uni.getStorageSync('user-num') === 5) {
			  this.num = uni.getStorageSync('user-num');
			}else{
				this.num = 3;
			}

			
			// this.apiset()
			try {
				const value = uni.getStorageSync('sk');
				// if (value) {
				// 	console.log(value);
				// 	this.api = value

				// }
			} catch (e) {
				// error
				console.log(e);
			}
			// uni.request({
			// 	url: this.apiurl,

			// 	 method: 'GET',
			// 	success: (res) => {
			// 		console.log(res);
			// 		this.apiadj = res.data
			// 	}
			// })

		},
		methods: {
			copyText(value) {
				uni.setClipboardData({
				data:value,
				success() {
					uni.showToast({
						title:'已复制到剪贴板',
						icon:'none',
						position:'top'
					        })
						}
					})
			},
			setsklocal(apikey) {
				uni.setStorage({
					key: 'sk',
					data: apikey,
					
					success: function(res) {
						console.log('success', res);
					}
				});
			},
			onChooseAvatar(e) {
				uni.setStorageSync('user-avatar',e.detail.avatarUrl);
				setTimeout(()=>{
						uni.navigateTo({
							url:'/pages/index/index'
						})
				},300)
				
			},
			clopop() {
				this.$refs.popup.close('center')
			},
			openpop() {
				this.$refs.popup.open('center')
			},
			 apiset() {
			 	// this.$refs.popup.close('center')
			 	this.apibut = 'api检测中,请稍等...'
			 	let data = JSON.stringify({
			 		body: "你好",
			// 		// openaikey: this.api
			 	})
			 	uni.request({
			 		url: this.apiurl ,
			 		data: data,
			 		method: 'POST',
			 	})

			 },
			sendMsg() {
				if(this.num<=0){
					this.msgList.push({
						"msg": "您的次数已用尽，分享可获得次数。",
						"my": false
					})
					this.rewardedVideoAd.show()
					return
				}
				// 消息为空不做任何操作
				if (this.msg == "") {
					return 0;
				}
				if (this.msgLoad == true) {
					this.$u.toast('请先配置api再进行使用')
					return 0
				}
				this.msgContent += ('YOU:' + this.msg + "\n")
				let data = JSON.stringify({
					body: this.msg
				})
				console.log(data)
				this.isRequesting=true;
				// this.sentext = '请求中'
				this.msgList.push({
					"msg": this.msg,
					"my": true
				})
				this.msgContent += ('YOU:' + this.msg + "\n")
				console.log(this.msgContent);
				this.msgLoad = true
				// 清除消息
				this.msg = ""
				uni.request({
					url: this.apiurl,
					data: data,
					method: 'POST',
					success: (res) => {
						//if (res.data.code == 200) {
							let text = res.data.choices[0].text.replace("openai:", "").replace("openai：", "").replace(/^\n|\n$/g, "")
							console.log(text);
							this.msgList.push({
								"msg": text,
								"my": false
							})
							this.isRequesting=false;
							this.num--
							this.msgContent += (text + "\n")
							this.msgLoad = false
							uni.setStorageSync('user-num',this.num);
							// this.sentext = `发送(${this.num}次)`
							this.scrollToBottom()
						//} else {
						//	this.apibut = '连接失败，请检查apikey后重试'
						//	this.apisucc = false
						//}
					},
					fail: (err) => {
						console.log(3344444,'失败');
					}
				})

			},
			scrollToBottom(){
				const that=this;
				setTimeout(()=>{
					that.scrollTop++
				},500)
			},
		}
	}
</script>

<style>
	page {
		height: 100%;
	}
	.advertising{
		
	}
	.bg {
		/* overflow: scroll; */
		/* background: url('../../static/6.png') no-repeat;
		background-size: 100% 100%; */
		width: 100%;
		height: 100%;
	}

	.bg-img {
		width: 100%;
		height: 100%;
		position: fixed;
	}

	.userinfo {
		animation: oneshow 0.8s ease 1;
		display: flex;
		flex-direction: column;
		align-items: flex-end;
		padding-right: 20rpx;
		margin-top: 20rpx;
	}

	.usermsg {
		margin-left: 20rpx;
		padding: 17rpx 20rpx;
		border-radius: 35rpx;
		background-color: #f9f9f9 !important;
		// margin-top: 20rpx;
	}

	.my-info {
		display: flex;
		align-items: center;
	}

	.aiinfo {
		display: flex;
		flex-direction: row;
		align-items: center;
		margin-left: 20rpx;
		margin-top: 20rpx;
		animation: oneshow 0.8s ease 1;
	}

	.chat-img {
		overflow: hidden;
		border-radius: 50%;
		/* width: 100rpx;
		height: 100rpx; */
		background-color: #f7f7f7;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}

	.aimsg {
		display: flex;
		flex-direction: column;
		justify-content: center;
		margin-left: 20rpx;
		padding: 17rpx 20rpx;
	}

	.flex-column-center {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: fixed;
		bottom: 0px;
		width: 100%;
	}

	.inpubut {
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
		/* background-color: #f9f9f9; */
		width: 100%;
		height: 110rpx;
		font-size: 40rpx;
	}

	.dh-input {
		width: 90%;
		height: 80rpx;
		border-radius: 100rpx;
		padding-left: 40rpx;
		margin-left: 20rpx;

		background-color: #f0f0f0;
	}

	.my-neirong-sm {
		font-size: 32rpx;
		color: #616161;
	}

	.btn {
		height: 80rpx;
		line-height: 80rpx;
		width: 35%;
		background: linear-gradient(to right, #008FFF, #29C8FC);
		color: #ffffff;
		font-size: 32rpx;
		;
		border-radius: 2500px;
		margin: 0 20rpx;

	}

	.popcls {
		width: 80vw;
		height: 40vh;
		background: white;
		border-radius: 20rpx;
		display: flex;
		flex-direction: column;
		justify-items: center;
		align-items: center;



	}
</style>
