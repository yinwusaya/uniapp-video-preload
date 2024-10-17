<template>
	<view class="content">
		<view id="user-click"></view>
		<swiper
			class="swiper"
			circular
			:indicator-dots="indicatorDots"
			:autoplay="autoplay"
			:interval="interval"
			@change="handleChange"
			:duration="duration"
			vertical
		>
			<swiper-item v-for="(item, index) in showList" :key="index">
				<view style="position: relative;width: 750rpx;height: 100vh" ref="videoContainerRef">
					<!-- 当前浏览的视频需要立即加载 -->
					<video
						v-if="activeIndex == index && !item.error"
						:id="`video_${index}`"
						ref="videoRef"
						style="width: 750rpx; height: 100%"
						:src="item.url"
						:poster="item.poster"
						@progress="videoProgress"
						@error="e => onVideoError(e,index)"
						autoplay
						object-fit="fill"
					></video>
					<!-- 不是正在浏览的视频，需要当前播放的视频加载完成，以及封面加载预加载完成，在进行加载 -->
					<video
						v-else-if="!item.error"
						:id="`video_${index}`"
						ref="videoRef"
						style="width: 750rpx; height: 100%"
						:src="showList.find(v => !v.posterLoad) ? '' : item.url"
						@progress="videoProgress"
						@error="e => onVideoError(e,index)"
						:poster="item.poster"
						autoplay
						object-fit="fill"
					></video>
					<image
						@load="imageloaded(index)"
						style="position: absolute;top: 0;bottom: 0;right: 0;left: 0;"
						v-show="activeIndex != index || item.error"
						:src="item.poster"
						class="video-poster"
						mode="scaleToFill"
					></image>
				</view>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
import { nextTick } from 'vue';
export default {
	data() {
		return {
			title: "Hello",
			indicatorDots: false,
			autoplay: false,
			interval: 2000,
			duration: 500,
			videoList: [
				{
					id:'1',
					url:"/static/1.mp4",
					poster: '/static/1.png'
				},
				{
					id:'2',
					url:"/static/2.mp4",
					poster: '/static/2.png'
				},
				{
					id:'3',
					url:"/static/3.mp4",
					poster: '/static/3.png'
				},
				{
					id:'4',
					url:"/static/4.mp4",
					poster: '/static/4.png'
				},
				{
					id:'5',
					url:"/static/5.mp4",
					poster: '/static/5.png'
				},
				// "/static/6.mp4",
				// "/static/7.mp4",
			],
			showList: [],
			videoContexts: [],
			activeIndex: 0,
			originIndex: 0, // 所有视频列表中的位置
		};
	},
	onLoad() {
		this.videoContexts = [
			uni.createVideoContext("video_0", this),
			uni.createVideoContext("video_1", this),
			uni.createVideoContext("video_2", this), 
		];
		nextTick(() => {
			this.$refs.videoRef.forEach((v,index) => {
				v.$el.addEventListener('click',() => {
					this.videoContexts[index].play()
				})
			})
		})

		this.showList = this.videoList.slice(0,3)
	},
	methods: {
		onVideoError(e,index) {
			uni.showToast({
				title: '网络连接失败',
				icon:'error'
			})
			this.showList[index].error = true
		},
		videoProgress(e){
		},
		imageloaded(index) {
			this.showList[index].posterLoad = true
		},
		handleChange(e) {
			// 判断向前向后滚动
			const { current } = e.detail
			if (current - this.activeIndex == 1 || current - this.activeIndex == -2) {
				// 向前滚动
				this.originIndex += 1
			} else {
				this.originIndex = this.originIndex == 0 ? 0 : this.originIndex - 1
			}
			
			this.activeIndex = current
			console.log('视频位置', this.originIndex)
			
			if (this.originIndex == this.videoList.length - 1){
				this.videoList.push(...this.videoList)
				// TODO: loadmore
			}
			
			// 暂停所有播放的视频,跳转到0s
			this.videoContexts.forEach(v => {
				v?.pause()
				v?.seek(0)
			})
			// 播放当前位置的视频
			setTimeout(() => {
				this.videoContexts[current].play()
			},100)

			// const videoList = []

			// if (current == 1) {
			// 	videoList[0] = this.originIndex == 0 ? this.videoList[0] : this.videoList[this.originIndex - 1]
			// 	videoList[1] = this.originIndex == 0 ? this.videoList[1] : this.videoList[this.originIndex]
			// 	videoList[2] = this.originIndex == 0 ? this.videoList[2] : this.videoList[this.originIndex + 1]
			// } else if (current == 0) {
			// 	videoList[0] = this.originIndex == 0 ? this.videoList[1] : this.videoList[this.originIndex]
			// 	videoList[1] = this.originIndex == 0 ? this.videoList[0] : this.videoList[this.originIndex + 1]
			// 	videoList[2] = this.originIndex == 0 ? this.videoList[2] : this.videoList[this.originIndex - 1]
			// } else if (current == 2) {
			// 	videoList[2] = this.originIndex == 0 ? this.videoList[2] : this.videoList[this.originIndex]
			// 	videoList[0] = this.originIndex == 0 ? this.videoList[1] : this.videoList[this.originIndex + 1]
			// 	videoList[1] = this.originIndex == 0 ? this.videoList[0] : this.videoList[this.originIndex - 1]
			// }
			
			// console.log('播放列表',videoList)
			// this.showList = videoList
		},
	},
};
</script>

<style>
.uni-margin-wrap {
	width: 690rpx;
	width: 100%;
}
.swiper {
	height: 100vh;
}
.swiper-item {
	display: block;
	height: 100vh;
	text-align: center;
}
.swiper-list {
	margin-top: 40rpx;
	margin-bottom: 0;
}
.uni-common-mt {
	margin-top: 60rpx;
	position: relative;
}
.info {
	position: absolute;
	right: 20rpx;
}
.uni-padding-wrap {
	width: 550rpx;
	padding: 0 100rpx;
}
.video-poster {
	background-color: #000;
	position: absolute;
	width: 100%;
	height: 100%;
}
</style>
