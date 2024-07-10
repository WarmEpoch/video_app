<template>
	<view id="home">
		<view class="header">
			<navigator url="info" hover-class="none"><image :src="token.avatar" class="avatar"></image></navigator>
		</view>
		<view class="main">
			<view class="form">
				<view class="input">
					<input type="search" v-model="search" :placeholder="video.place" @confirm="Searchs" @input="SearchI"/>
					<image src="../static/icon/circle-close.svg" class="close" v-show="search" @click="search = ''"></image>
				</view>
				<image src="../static/icon/search.svg" @click="Searchs" class="search"></image>
			</view>
			<view class="suggest" v-show="suggest">
				<text>搜索建议</text>
				<view class="all">
					<view class="scroll">
						<button v-for="(history,index) of suggest" :key="index" @click="Search(history)">{{ history }}</button>
					</view>
				</view>
			</view>
			<view class="notices" v-show="video.notices">
				<text>公告</text>
				<view class="all">
					<view class="scroll">
						<text>{{ video.notices }}</text>
					</view>
				</view>
			</view>
			<view class="history" v-show="historys.length">
				<view class="top">
					<text>历史搜索</text>
					<image @click="Delete" src="../static/icon/delete.svg"></image>
				</view>
				<view class="all">
					<view class="scroll">
						<button v-for="(history,index) of historys" :key="index" @click="Search(history)">{{ history }}</button>
					</view>
				</view>
			</view>
			<view class="hots" v-show="hots.movie">
				<text>热门电影</text>
				<view class="movie">
					<view class="scroll">
						<button v-for="movie of hots.movie" :key="movie.id" @click="Search(movie.title)">{{ movie.title }}</button>
					</view>
				</view>
			</view>
			<view class="hots" v-show="hots.tv">
				<text>热门电视剧</text>
				<view class="tv">
					<view class="scroll">
						<button v-for="tv of hots.tv" :key="tv.id" @click="Search(tv.title)">{{ tv.title }}</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				video: JSON.parse(plus.storage.getItem('video')) || {
					place: '开端',
					notices: false,
				},
				hots: {
					movie: JSON.parse(plus.storage.getItem('movie')),
					tv: JSON.parse(plus.storage.getItem('tv'))
				},
				token: {},
				search: '',
				historys: [],
				suggest: false,
			}
		},
		onLoad() {
			this.GetHotsMovie()
			this.GetHotsTv()
			// this.GetImmersVideo()
		},
		onShow() {
			if(plus.storage.getItem("historys")) this.historys = JSON.parse(plus.storage.getItem("historys"))
			this.token = JSON.parse(plus.storage.getItem("token")) || {avatar: '../static/icon/video.svg'}
			this.token.avatar = this.token.avatar || '../static/icon/video.svg'
		},
		methods: {
			
			Search(keyword = this.search){
				if(!this.historys || this.historys[0] != keyword){
					this.historys.unshift(keyword)
					let _historys = Array.from(new Set(this.historys))
					this.historys = _historys
					plus.storage.setItem('historys', JSON.stringify(_historys))
				}
				this.video.place = keyword
				this.search = ''
				this.suggest = false
				uni.navigateTo({
					url: 'list?s=' + keyword
				})
			},
			
			Searchs(){
				this.search.trim().length > 0 ? this.Search() : this.Search(this.video.place)
			},
			
			GetHotsMovie(){
				uni.request({
					url: 'https://movie.douban.com/j/search_subjects',
					data: {
						type: 'movie',
						tag: '热门',
						page_limit: 10,
						page_start: 0
					}
				}).then(res => {
					this.hots.movie = res.data?.subjects
					plus.storage.setItem('movie',JSON.stringify(res.data?.subjects))
				})
			},
			
			
			GetHotsTv(){
				uni.request({
					url: 'https://movie.douban.com/j/search_subjects',
					data: {
						type: 'tv',
						tag: '热门',
						page_limit: 10,
						page_start: 0
					},
					success: res =>{
						this.hots.tv = res.data?.subjects
						plus.storage.setItem('tv',JSON.stringify(res.data?.subjects))
					}
				})
			},
			
			SearchI(){
				if(this.search.trim().length > 0){
					uni.request({
						url: 'https://movie.douban.com/j/subject_suggest',
						data:{
							q: this.search
						},
						success: res => {
							if(res.data.length){
								let data = []
								res.data.forEach(val => data.push(val.title))
								this.suggest = new Set(data)
							}else{
								this.suggest = false
							}
						}
					})
				}else{
					this.suggest = false
				}
			},
			
			GetImmersVideo(){
				uni.request({
					url: 'https://v.immers.icu/api/updata',
					data:{
						t: Math.random()
					},
					method: 'GET',
					success: res => {
						this.video = res.data
						plus.storage.setItem('video',JSON.stringify(res.data))
					},
					fail: res => {
						uni.getNetworkType({
							success: res => {
								if(res.networkType == 'none'){
									plus.nativeUI.toast("无连接网络")
								}else{
									plus.nativeUI.toast("无网络连接")
								}
							}
						})
					},
					complete: async (res) => {
						if(this.video?.new?.version && this.video.new.version > plus.runtime.versionCode){
							uni.showModal({
								title: this.video.new.title,
								content: this.video.new.digest,
								showCancel: this.video.new.showCancel,
								success: e => {
									if(!this.video.new.showCancel && !e.confirm){
										plus.runtime.quit()
									}
									if (e.confirm) {
										if(uni.getSystemInfoSync().platform == 'android'){
											plus.runtime.openURL(url)
										}else{
											plus.runtime.openURL('https://v.immers.icu')
										}
									}
								}
							})
							let url = await this.Lanzou(this.video.new.download)
						}
					}
				})
			},
			
			Delete(){
				this.historys = []
				plus.storage.removeItem('historys')
			},
			
			async Lanzou(url){
				const res = await uni.request({
				    url: url,
				})
				return `https://develope.lanzoug.com/file/${[...res.data.matchAll(/'(\?.*)';/g)][0][1]}`
			}
			
		}
	}
</script>

<style>
	
	.main{
		display: flex;
		flex-flow: column wrap;
	}
	
	.main .form{
		display: flex;
		justify-content: space-between;
		width: 100%;
		padding: 6%;
		margin-bottom: 6%;
		background: #fff;
		border-radius: 1rem;
		box-sizing: border-box;
	}
	
	.main .form .input{
		flex: 1;
		position: relative;
		display: flex;
		align-items: center;
	}
	
	.main .form input{
		width: 100%;
		height: 100%;
		color: #666;
		font-size: 1rem;
	}
	
	.main .form .close{
		width: 1.3rem;
		margin-left: 1rem;
		height: unset;
		aspect-ratio: 1 / 1;
		position: absolute;
		right: 0;
	}
	
	
	.main .form .search{
		width: 1.8rem;
		margin-left: 1rem;
		height: unset;
		aspect-ratio: 1 / 1;
	}
	
	.main .form .search:active{
		opacity: .6;
	}
	
	
	.main .hots,.main .history,.main .suggest,.main .notices{
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
	
	.main .hots text,.main .history text,.main .suggest text,.main .notices>text{
		font-size: 1.1rem;
		font-weight: 500;
	}
	
	.main .hots .movie,.main .hots .tv,.main .history .all,.main .suggest .all,.main .notices .all{
		display: flex;
		overflow: hidden;
		width: 100%;
		overflow-x: auto;
		margin-top: .4rem;
	}
	
	.main .hots .scroll,.main .history .scroll,.main .suggest .scroll,.main .notices .scroll{
		display: flex;
	}
	
	.main .hots button,.main .history button,.main .suggest button{
		white-space: nowrap;
		padding: .4rem 1rem;
		margin: .3rem .6rem .3rem 0;
		line-height: unset;
		background: #f5f5f5;
		border-radius: 1rem;
		border: none;
		font-size: .8rem;
	}
	
	.main .hots button:last-child,.main .history button:last-child,.main .suggest button:last-child{
		margin-right: 0;
	}
	
	.main .hots button::after,.main .history button::after,.main .suggest button::after{
		content: unset;
	}
	
	.main .history image{
		width: unset;
		height: 1rem;
		aspect-ratio: 1 / 1;
	}
	
	.main .history image:active{
		opacity: .8;
	}
	
	.main .history .top{
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	
</style>
