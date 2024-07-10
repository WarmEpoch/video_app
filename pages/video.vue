<template>
</template>

<script>
	export default {
		data() {
			return {
				webView: false,
				show: true,
				hide: false
			}
		},
		async onLoad(option) {
			
			this.webView = plus.webview.create('https://www.aliyundrive.com/protocol/permission','play',{
				// https://www.alipan.com/protocol/permission
				// https://www.aliyundrive.com/protocol/permission
				"scrollIndicator": "none",
				"background": "#f5f5f5"
			},option)
			
			this.webView.hide()
			this.webView.setCssText('#root{display:none}')
			this.webView.appendJsFile('/static/js/new.js')
			this.webView.onloaded = () => {
				console.log('webview加载结束')
				this.show = 'real.Show()'
				this.hide = 'real.Hide()'
			}
			this.$scope.$getAppWebview().append(this.webView)
			console.log('webview添加完成')
		},
		onShow(){
			this.webView.evalJS(this.show)
		},
		onHide(){
			this.webView.evalJS(this.hide)
		},
		methods: {
			
		}
	}
</script>

<style>
	
</style>
