<template>
	<view id="list">
		<view class="header">
			<navigator url="info" hover-class="none"><image :src="token.avatar" class="avatar"></image></navigator>
		</view>
		<view class="main">
			<view v-for="list of lists" :key="list[1]" @click="Open(list[3],list[1],list[2])" class="list">
				<view :style="`background-image: url(${list[2]}), url(../static/img/cover.gif);`" mode="aspectFill" @longpress="Opens(list[3],list[1])" class="cover"></view>
				<view class="info">
					<text class="title">{{list[3]}}</text>
					<uni-rate class="rate" :disabled="true" disabledColor="#ffca3e" :value="list[4] / 2" />
				</view>
			</view>
			<view v-show="error" class="error">
				<icon type="warn" size="66" color="#aaa"></icon>
				<text>这样真的出不来。。。</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token: {},
				keyword: String,
				part: 0,
				lists: [],
				played: [],
				error: false,
			}
		},
		onShow(){
			this.played = JSON.parse(plus.storage.getItem("played")) || []
			this.token = JSON.parse(plus.storage.getItem("token")) || {avatar: '../static/icon/video.svg'}
			this.token.avatar = this.token.avatar || '../static/icon/video.svg'
			this.token.nick_name = this.token.nick_name || 'Immers'
		},
		onLoad: async function(option) {
			this.keyword = option.s
			uni.showLoading({
				title: '加载中',
				mask: true
			})
			const lists = await this.Searchs(this.keyword, this.part)
			uni.hideLoading()
			if(!lists.length){
				this.error = true
				return false
			}
			this.lists = lists
		},
		methods: {
			
			async Searchs(q, p) {
			
				const Searchs = await uni.request({
					url: 'https://m.douban.com/j/search/',
					method: 'GET',
					data: {
						q,
						t: '1002',
						p
					},
				}).then(res => {
					if(!res.data.count) return []
					return [...res.data.html.matchAll(/<li>.*?<a href="\/movie\/subject\/(\d*)\/">.*?<img src=\"(.*?)"\/>.*?<span class=\"subject-title\">(.*?)<\/span>.*?<span>(.*?)<\/span>.*?<\/li>/gs)]
				}).catch(res => {
					return []
				})
			
				return Searchs
			
			},
			
			async GetUrl(id) {
				const GetUrl = await uni.request({
					url: `https://v.immers.icu/api/get?id=${id}`,
					method: 'GET',
				}).then(res => {
					return res.data?.url
				}).catch(res => {
					return false
				})
			
				return GetUrl
			
			},
			
			async Id(title, db, img) {
				let keyword = title.length > 5 ? title.substring(0,4) : title
				const Id = await uni.request({
					method: 'post',
					url: 'https://cmn.yydshd.com/api/posts',
					data: {
						limit: 10,
						skip: 0,
						keyword,
						category_id: -1,
					}
				}).then(res => {
					for (let i in res.data.data.list) {
						let _img = res.data.data.list[i].cover.includes('douban') ? res.data.data.list[i].cover.match(/public\/(.*).jpg/)[1] : 'not_img'
						if (res.data.data.list[i].db_id == db || _img == img) {
							return res.data.data.list[i].id.toString()
						}
					}
					return false
				})
				
				return Id
			},

			async Link(id) {
				const Token = await uni.request({
					url: "https://api.immers.icu/api/Video/token",
					method: 'GET',
				}).then(res => {
					return res.data
				})
				
				const Link = await uni.request({
					method: "post",
					url: "https://cmn.yydshd.com/api/post-info",
					header: {
						token: Token,
					},
					data: {
						id,
					},
				}).then(res => {
					for (let i in res.data.data.links) {
						if (res.data.data.links[i].name.indexOf('阿里网盘') != -1) {
							let val = res.data.data.links[i].item[0].link.match(/.*\/s\/(.*)/)[1]
							return val
						}
					}
					return false
				})
				
				return Link

			},
			
			async Open(name, db, img) {
				uni.showLoading({
					title: '加载中',
					mask: true
				})
				
				if(true){
					uni.hideLoading()
					// uni.navigateTo({
					// 	url: `video?db=${db}&name=${name}&link=3VmFUKZCBVw`
					// })
					uni.navigateTo({
						url: `play`
					})
					return true
				}
				const _index = this.played.findIndex(res => res.db == db)
				if(_index >= 0){
					uni.hideLoading()
					uni.navigateTo({
						url: `video?db=${db}&name=${name}&link=${this.played[_index].link}`
					})
					return true
				}
				
				const GetUrl = await this.GetUrl(db)
				
				if(GetUrl){
					uni.hideLoading()
					uni.navigateTo({
						url: `video?db=${db}&name=${name}&link=${GetUrl}`
					})
					return true
				}
				
				
				const _img = img.match(/public\/(.*).jpg/)[1]
				const Id = await this.Id(name, db, _img)
				if(!Id){
					uni.hideLoading()
					uni.showToast({
						title: '未找到片源',
						icon: 'error',
						duration: 1000
					})
					return false
				}
				const Link = await this.Link(Id)
				if(!Link){
					uni.hideLoading()
					uni.showToast({
						title: '无播放源',
						icon: 'error',
						duration: 1000
					})
					return false
				}
				uni.hideLoading()
				uni.navigateTo({
					url: `video?db=${db}&name=${name}&link=${Link}`
				})
			},
			
			
			Opens(name, db){
				uni.showModal({
					title: '自定义链接',
					content: '将阿里云盘分享链接粘贴至此，共享你的资源。',
					editable: true,
					confirmText: '确定',
					success: async (res) => {
						if(res.content){
							let link = [...res.content.matchAll(/.*aliyundrive.com\/s\/([0-9A-Za-z]{11}).*/g)]
							if(link[0]){
								const CreateUrl = await this.CreateUrl(db,link[0][1])
								if(CreateUrl){
									plus.nativeUI.toast("共享完成")
								}else{
									plus.nativeUI.toast("请重试")
								}
							}else{
								plus.nativeUI.toast("链接无效")
							}
						}
					}
				})
			},
			
			async CreateUrl(id,url){
				
				const CreateUrl = await uni.request({
					url: `https://v.immers.icu/api/create?id=${id}&url=${url}`,
					method: 'GET',
				}).then(res => {
					return res.data
				}).catch(res => {
					return false
				})
			
				return CreateUrl
			}
			
			
		}
	}
</script>

<style>
	.main{
		display: flex;
		flex-flow: column wrap;
	}
	
	.main .list{
		width: 100%;
		padding: 6%;
		background: #fff;
		border-radius: 1rem;
		box-sizing: border-box;
		margin-bottom: 6%;
		display: flex;
		align-items: center;
	}
	
	.main .list .cover{
		width: 40%;
		height: unset;
		aspect-ratio: 270 / 383;
		overflow: hidden;
		background-size: cover;
		background-repeat: no-repeat;
		background-position: center;
		border-radius: .3rem;
	}
	
	.main .list .info{
		width: 60%;
		display: flex;
		padding-left: 1rem;
		flex-flow: column wrap;
		box-sizing: border-box;
	}
	
	.main .list .title{
		font-weight: bold;
		font-size: 1.2rem;
		line-height: 1.6rem;
		margin-left: .2rem;
		margin-bottom: .3rem;
	}
	
	.main .list .rate .uni-icons{
		font-size: 1.5rem!important;
	}
	
	.main .error{
		text-align: center;
		display: flex;
		flex-flow: column wrap;
		align-items: center;
		justify-content: center;
		color: #bbb;
		min-height: 76vh;
		line-height: 1.6rem;
	}
	
	
</style>
