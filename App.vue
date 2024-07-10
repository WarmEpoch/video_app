<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: async function() {
			if (process.env.NODE_ENV === 'development') {
				return false
			}
			let args = [...plus.runtime.arguments.matchAll(/:\/\/(.*)\/(.*)\/(.*)/g)][0]
			if (args) {
				uni.showLoading({
					title: '加载中',
					mask: true
				})
				plus.runtime.arguments = false
				let db = parseInt(args[1], 32)
				let name = await this.GetName(db)
				uni.hideLoading()
				uni.navigateTo({
					url: `/pages/video?db=${db}&name=${name}&link=${args[2]}&part=${args[3]}`
				})
			} else {
				let clip = await uni.getClipboardData().then(res => res.data).catch(res => '')
				if (clip.search(/沐享/) >= 0) {
					let args = [...clip.matchAll(/icu\/(.*)\/(.*)\/(.*)\s/g)][0]
					if (args[2] == plus.storage.getItem('copy')) return false
					let db = parseInt(args[1], 32)
					let name = await this.GetName(db)
					this.Share(db, name, args[2], args[3])
				}
			}
		},
		onHide: function() {
			console.log('App Hide')
		},
		methods: {

			async Share(db, name, link, part) {
				uni.showModal({
					title: name,
					content: '如是你要打开的密钥，请点击确定按钮。',
					success: function(res) {
						if (res.confirm) {
							uni.navigateTo({
								url: `video?db=${db}&name=${name}&link=${link}&part=${part}`
							})
						}
					},
					complete: function() {
						plus.storage.setItem('copy', link)
					}
				})
			},

			async GetName(db) {
				const Name = await uni.request({
					url: `https://movie.douban.com/subject/${db}/output_card`,
					method: 'GET',
					timeout: 5000,
				}).then(res => {
					return [...res.data.matchAll(/<h1>(.*)<\/h1>/g)][0][1]
				})

				return Name
			},

		}
	}
</script>

<style>
	/* #ifndef APP-PLUS-NVUE */
	page {
		background: #f5f5f5;
		font-family: -apple-system-font, Helvetica Neue, sans-serif;
	}


	.header {
		position: sticky;
		top: 0;
		width: 100%;
		display: flex;
		justify-content: space-between;
		backdrop-filter: saturate(180%) blur(20px);
		background-color: rgba(255, 255, 255, 0.72);
		box-sizing: border-box;
		z-index: 10;
		padding: calc(var(--status-bar-height) + 3%) 6% 3% 6%;
	}

	.header .avatar {
		width: 2.3rem;
		height: unset;
		aspect-ratio: 1 / 1;
		border-radius: 50%;
	}

	.header image {
		width: 1.8rem;
		height: unset;
		aspect-ratio: 1 / 1;
		border-radius: 50%;
	}

	image {
		display: block;
	}


	.main {
		padding: 6%;
		box-sizing: border-box;
	}

	:focus-visible {
		outline: none;
	}
	/* #endif */
</style>