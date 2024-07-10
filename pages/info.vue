<template>
	<view id="info">
		<view class="header">
			<navigator open-type="navigateBack" hover-class="none">账户</navigator>
		</view>
		<view class="main">
			<button class="info" @click="Info">
				<image :src="token.avatar"></image>
				<text>{{ token.nick_name }}</text>
			</button>
			<view class="played" v-show="played.length">
				<view class="top">
					<text>播放历史</text>
					<image @click="Delete" src="../static/icon/delete.svg"></image>
				</view>
				<view class="all">
					<view class="scroll">
						<button v-for="(history,index) of played" :key="index" @click="Open(history.db,history.name,history.link)">{{ history.name }}</button>
					</view>
				</view>
			</view>
			<view class="func" @click="ClearStorage()">清除数据</view>
			<view class="func" @click="ClearUser()" v-show="token.user_id">退出登录</view>
		</view>
		<view class="footer">
			Copyright © 2017 - {{ new Date().getFullYear() }} <text @click="Contact()">Immers</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token: {},
				played: [],
			}
		},
		onShow() {
			this.Show()
		},
		methods: {
			Show(){
				this.played = JSON.parse(plus.storage.getItem("played")) || []
				this.token = JSON.parse(plus.storage.getItem("token")) || {avatar: '../static/icon/video.svg',nick_name: '未登录'}
				this.token.avatar = this.token.avatar || '../static/icon/video.svg'
				this.token.nick_name = this.token.nick_name || 'Immers'
			},
			
			ClearStorage(){
				uni.showModal({
					title: '确定要清理数据吗？',
					content: '这包括历史记录、播放记录、ShareToken等，APP缓存数据可以减少网络请求。',
					success: async (res) => {
						if (res.confirm) {
							const Info = this.token
							plus.storage.clearAsync(function(){
								if(Info.user_id){
									plus.storage.setItemAsync('token',JSON.stringify(Info),function(){
										plus.runtime.restart()
									})
								}else{
									plus.runtime.restart()
								}
							})
						}
					}
				})
			},
			
			ClearUser(){
				uni.showModal({
					title: '确定要退出登录吗？',
					content: '这将退出阿里云盘的登录状态，会使程序无法正常获取到链接，程序将几近崩溃状态。',
				}).then(res => {
					if (res.confirm) {
						uni.showLoading({
							title: '正在退出',
							mask: true,
						})
						let out = plus.webview.create('https://www.aliyundrive.com/sign/out')
						out.onloaded = () => {
							plus.storage.removeItemAsync('token',() => {
								this.Show()
								uni.hideLoading()
							})
						}
					}
				})
			},
			
			
			Codec(e){
				this.config.codec = e.detail.value
				plus.storage.setItem('config',JSON.stringify(this.config))
			},
			
			Strategy(e){
				this.config.strategy = e.detail.value
				plus.storage.setItem('config',JSON.stringify(this.config))
			},
			
			Contact(){
				plus.runtime.openURL('https://v.immers.icu')
				// plus.gallery.save('https://shp.qpic.cn/collector/1523230910/54a8d580-ce74-4faf-8245-f4ac6272e9bd/0',function(){
				// 	plus.nativeUI.toast("二维码已保存")
				// 	plus.runtime.launchApplication({pname:"com.tencent.mm",action:'weixin://'})
				// })
			},
			
			Open(db,name,link){
				// uni.navigateTo({
				// 	url: `play?db=${db}&name=${name}&link=${link}`
				// })
				uni.navigateTo({
					url: `video?db=${db}&name=${name}&link=${link}`
				})
			},
			
			Info(){
				if(!this.token.user_id){
					uni.navigateTo({
						url: 'webview?call=get'
					})
					return
				}
				uni.showModal({
					title: '使用贴士',
					content: '可使用阿里云盘修改自己的昵称与头像，如需更新用户信息点击更新按钮。',
					confirmText: '更新',
					success: async (res) => {
						if (res.confirm) {
							uni.showLoading({
								title: '更新中',
								mask: true
							})
							const Info = await this.GetInfo()
							this.token = {...this.token , Info }
							plus.storage.setItem('token',JSON.stringify(this.token))
							this.token.avatar = this.token.avatar || '../static/icon/video.svg'
							this.token.nick_name = this.token.nick_name || 'Immers'
							uni.hideLoading()
						}
					}
				})
			},
			
			async GetInfo(){
				const Info = await uni.request({
					url: 'https://api.aliyundrive.com/adrive/v2/user/get',
					method: 'POST',
					data:{
						
					},
					header: {
						authorization: `${this.token.token_type} ${this.token.access_token}`
					}
				}).then(res => {
					return res.data
				}).catch(() => {})
				
				return Info
			},
			
			Delete(){
				this.played = []
				plus.storage.removeItem("played");
			}
			
		}
	}
</script>

<style>
	.header navigator{
		font-size: 1.3rem;
		height: 2.3rem;
		line-height: 2.3rem;
		font-weight: bold;
	}
	
	.main .info{
		width: 100%;
		padding: 6%;
		background: #fff;
		border-radius: 1rem;
		box-sizing: border-box;
		margin-bottom: 6%;
		display: flex;
		align-items: center;
	}
	
	.main .info::after{
		content: unset;
	}
	
	.main .info image{
		width: 23%;
		height: unset;
		aspect-ratio: 1 / 1;
		border-radius: 50%;
	}
	
	.main .info text{
		font-size: 1.6rem;
		margin-left: 1rem;
	}
	
	.main .played{
		display: flex;
		flex-flow: column wrap;
		width: 100%;
		padding: 6%;
		margin-bottom: 6%;
		background: #fff;
		border-radius: 1rem;
		box-sizing: border-box;
		overflow: hidden;
	}
	
	.main .played text{
		font-size: 1.1rem;
		font-weight: 500;
	}
	
	.main .played image{
		width: unset;
		height: 1rem;
		aspect-ratio: 1 / 1;
	}
	
	.main .played image:active{
		opacity: .8;
	}
	
	.main .played .top{
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	
	.main .played .all{
		display: flex;
		overflow: hidden;
		width: 100%;
		overflow-x: auto;
		margin-top: .4rem;
	}
	
	.main .played .scroll{
		display: flex;
	}
	
	.main .played button{
		white-space: nowrap;
		padding: .4rem 1rem;
		margin: .3rem .6rem .3rem 0;
		line-height: unset;
		background: #f5f5f5;
		border-radius: 1rem;
		border: none;
		font-size: .8rem;
	}
	
	.main .played button:last-child{
		margin-right: 0;
	}
	
	.main .played button::after{
		content: unset;
	}
	
	.main .func,.main .control{
		display: flex;
		flex-flow: column wrap;
		width: 100%;
		padding: 6%;
		background: #fff;
		border-radius: 1rem;
		box-sizing: border-box;
		margin-bottom: 6%;
	}
	
	.main .control .label{
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: .3rem 0;
		font-size: 1rem;
	}
	
	.main .label radio-group{
		display: flex;
		align-items: center;
	}
	
	.main .label radio{
		transform: scale(.8);
	}
	
	.main .label label{
		display: flex;
		align-items: center;
		margin-right: 1rem;
	}
	
	.main .label label:last-child{
		margin-right: 0;
	}
	
	.main .func:active{
		opacity: .8;
	}
	
	.main .func button{
		width: 100%;
		background: none;
		border: none;
		border-top: solid 1px #eee;
		border-radius: unset;
		color: #333;
		font-size: 1rem;
	}
	
	.main .func button:first-child{
		border-top: none;
	}
	
	.main .func button::after{
		content: unset;
	}
	
	.footer{
		text-align: center;
		height: 6rem;
		font-size: .9rem;
		color: #bbb;
	}
	
	.footer text{
		text-decoration: underline;
	}
</style>
