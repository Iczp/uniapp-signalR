

<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
			uni.$on('receivemessage', e => {

				console.log('uni.$on:receivemessage', e)
				uni.showToast({
					title: `收到消息:${e}`
				})
			})
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>


<style>
	/*每个页面公共css */
</style>