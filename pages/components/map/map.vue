<template>
	<uni-drawer >
		<view style="padding:30rpx;">
			<u-action-sheet :list="list" v-model="show"></u-action-sheet>
			<canvas canvas-id="map" style="328px;height:582px;"></canvas>
		</view>
	</uni-drawer>
</template>

<script>
	export default {
		data() {
			return {
				list: [{
					text: '点赞',
					color: 'blue',
					fontSize: 28
				}, {
					text: '分享'
				}, {
					text: '评论'
				}],
				show: true
			}
		},
		mounted() {

		}
	}
</script>

<style>
</style>
