<template>
	<view>
		<web-view src="https://changelly.com/buy"></web-view>
	</view>
</template>

<script>
	export default {
		onLoad() {
			let pages = getCurrentPages()
			let page = pages[pages.length - 1];
			let currentWebview = page.$getAppWebview();
			let titleNView = currentWebview.getStyle().titleNView
			console.log('111111', titleNView)
		}
	};
</script>
