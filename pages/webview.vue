<template>
</template>

<script>
	export default {
		data() {
			return {
				webView: false
			}
		},
		onLoad(option){
			switch(option.call){
				case 'get':
					// plus.navigator.setUserAgent('Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.124 Safari/537.36 Edg/102.0.1245.44')
					// this.webView = plus.webview.create('https://www.aliyundrive.com/sign/in')
					plus.navigator.setUserAgent('Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36 Edg/121.0.0.0')
					this.webView = plus.webview.create('https://www.alipan.com/sign/in')
					this.webView.appendJsFile('/static/js/get_token.js')
					this.webView.setCssText('.home .download{display: none;}')
					break
				default:
					uni.navigateBack()
			}
			this.$scope.$getAppWebview().append(this.webView)
		}
	}
</script>

<style>

</style>
