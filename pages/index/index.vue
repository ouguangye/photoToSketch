<template>
	<view>
		<view class="upload_container" v-if="uploadSwitch" v-on:click="choosePhoto">
			<image src="../../static/image/add_photo.png" />
			<view class="text_container">
				<h4>选择图片</h4>
				<text class="detail">选择图片生成素描图片</text>
			</view>
		</view>

		<view class="icon_container" v-else>
			<image src="../../static/image/refresh.png" v-on:click="refresh" />
			<image src="../../static/image/download.png" v-on:click="save" />
			<image src="../../static/image/share.png" v-on:click="beginshare" />
		</view>

		<view class="uploaded_container">
			<h4 v-if="!uploadSwitch">上传的图片</h4>
			<canvas type="2d" id="first" canvas-id="first" v-show="!uploadSwitch"
				:style="{'width': canvasWidth + 'px','height': canvasHeight+'px'}"></canvas>
		</view>

		<view class="result_container">
			<h4 v-if="!uploadSwitch">生成的结果： </h4>
			<canvas type="2d" id="second" canvas-id="second" v-show="!uploadSwitch"
				:style="{'width': canvasWidth + 'px','height': canvasHeight+'px'}"></canvas>
		</view>

	</view>
</template>

<script>
	import cv from '@/uni_modules/zj-opencv';

	export default {
		data() {
			return {
				uploadSwitch: true,
				imageUrl: '',
				canvasWidth: 200,
				canvasHeight: 1300
			}
		},
		onLoad() {

		},
		methods: {
			refresh() {
				this.uploadSwitch = true;
				this.imageUrl = '';
			},
			choosePhoto() {
				const _this = this;
				uni.chooseImage({
					count: 1, //最多可以选择的图片张数，默认9
					sourceType: ['album'], //album 从相册选图，camera 使用相机，默认二者都有。如需直接开相机或直接选相册，请只使用一个选项
					sizeType: ['original'], //original 原图，compressed 压缩图，默认二者都有
					success(res) {
						console.log('选择图片完成', res)
						_this.uploadSwitch = false;
						_this.imageUrl = res.tempFilePaths[0];
						_this.processImage();
					},
					fail(err) {
						console.log('失败', err);
					},
					complete() {
						console.log('结束');
					},
				})
			},

			save() {
				console.log("save!!!");
				let that = this;
				uni.canvasToTempFilePath({
					canvasId: 'second',
					destWidth: that.canvasWidth,
					destHeight: that.canvasHeight,
					success: function(res) {
						console.log(res)
						uni.saveImageToPhotosAlbum({
							filePath: res.tempFilePath,
							success: function() {
								uni.showToast({
									title: '已经保存到相册',
									icon: 'none'
								})
							}
						});

						// 兼容h5的写法
						let oA = document.createElement('a');
						oA.download = 'sketch' + new Date().getTime();
						oA.href = res.tempFilePath;
						document.body.appendChild(oA);
						oA.click();
						oA.remove();
					},
					fail: function(error) {
						console.log(err);
						uni.openSetting({
							success: (response) => {
								if (!response.authSetting['scope.writePhotosAlbum']) {
									uni.showModal({
										title: '提示',
										content: '获取权限成功，再次点击图片即可保存',
										showCancel: false
									})
								} else {
									uni.showModal({
										title: '提示',
										content: '获取权限失败，无法保存',
										showCancel: false
									})
								}
							}
						})
					}
				})
			},

			//调起用户手机的服务商列表
			beginshare() {
				uni.showToast({
					title:'分享前建议先保存图片',
					icon:'none'
				})
				let that = this;
				uni.getProvider({
					service: 'share',
					success: function(res) {
						let provider = res.provider.splice(',')
						let newarr = [];
						for (let i in provider) {
							if (provider[i] == 'weixin') {
								newarr[0] = '分享到微信'
								newarr[1] = '分享到朋友圈'
							}
						}
						that.provider = newarr
						console.log(that.provider);
						that.share();
					}
				});
			},

			//对服务商列表进行排序分配相应的参数
			share() {
				var that = this
				uni.showActionSheet({
					itemList: that.provider,
					success: function(rest) {
						if (rest.tapIndex == 0) {
							that.sendimg("weixin", "WXSceneSession");
						} else if (rest.tapIndex == 1) {
							that.sendimg("weixin", "WXSenceTimeline");
						}
					},
					fail: function(res) {
						wx.showToast({
							title: '很抱歉，由于平台限制，分享失败',
							icon: 'none',
						})
						console.log(res.errMsg);
					}
				});
			},

			//用户从相册中选择图片进行分享
			sendimg(strProvider, strScene) {
				// 选取图片			
				uni.chooseImage({
					count: 1, 
					sizeType: ['compressed'],
					sourceType: ['album'],
					success: function(res) {
						uni.share({
							provider: strProvider,
							scene: strScene,
							type: 2,
							imageUrl: res.tempFilePaths[0],
							success: function(res) {
								uni.showToast({
									title: JSON.stringify(err),
									duration: 2000,
									icon: 'none'
								});
							},
							fail: function(err) {
								console.log("fail:" + JSON.stringify(err));
							}
						});


					},
				}) 	
			},

			async drawImage() {
				let that = this;
				return new Promise((resolve, reject) => {
					uni.createSelectorQuery()
						.select('#first')
						.fields({
								context: true,
								size: true,
								node: true
							},
							first => {
								console.log(first)
								const context = first.context;
								uni.createCanvasContext(context.id || context.canvasId);
								uni.getImageInfo({
									src: that.imageUrl,
									success: data => {
										console.log(data)
										that.canvasHeight = that.canvasWidth * data.height / data
											.width;
										context.drawImage(that.imageUrl, 0, 0, that.canvasWidth,
											that.canvasHeight);
										context.draw(false, () => {
											resolve();
										});
									}
								})

							})
						.exec();
				});
			},

			async processImage() {
				await this.drawImage();
				cv.then(async () => {
					const src = await cv.imread('first', cv.IMREAD_GRAYSCALE);

					// 灰度图像
					let I = new cv.Mat();
					cv.cvtColor(src, I, cv.COLOR_RGBA2GRAY);
					
					// 灰度图像取反
					let img_gray_inv = new cv.Mat();
					cv.bitwise_not(I, img_gray_inv);

					// 高斯模糊
					let img_blur = new cv.Mat();
					cv.GaussianBlur(img_gray_inv, img_blur, new cv.Size(21, 21), 0, 0);
					cv.bitwise_not(img_blur, img_blur);

					// 以混合模式为颜色减淡进行融合
					let L = new cv.Mat();
					cv.divide(I, img_blur, L, 255)
					
					// ostu 二值化处理 
					cv.threshold(L, L, 0, 255, cv.THRESH_BINARY | cv.THRESH_OTSU)

					await cv.imshow('second', L);

					I.delete();
					src.delete();
					img_gray_inv.delete();
					img_blur.delete();
					L.delete();
				})
			}

		}
	}
</script>

<style lang="scss">
	page {
		box-sizing: border-box;
		padding: 23rpx 23rpx;
	}

	.upload_container {
		background-color: #e9efeb;
		width: 95%;
		height: 360rpx;
		border-radius: 24rpx;
		box-sizing: border-box;
		padding: 40rpx;
		margin: 0 auto;
		margin-bottom: 46rpx;

		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: flex-start;

		$iconSize: 34px;

		image {
			width: $iconSize;
			height: $iconSize;
		}

		.text_container {
			.detail {
				font-size: 8rpx;
			}
		}
	}

	.icon_container {
		width: 100%;
		display: flex;
		justify-content: space-around;
		align-items: center;
		margin-bottom: 30rpx;

		$iconSize: 34px;

		image {
			width: $iconSize;
			height: $iconSize;
		}
	}

	.uploaded_container {
		width: 100%;
		margin-bottom: 40rpx;

		h4 {
			margin-bottom: 23rpx;
		}

		#first {
			margin: 0 auto;
			//width: v-bind(canvasWidth) px !important;
			//height: v-bind(canvasHeight) px !important;
		}
	}

	.result_container {
		width: 100%;

		h4 {
			margin-bottom: 23rpx;
		}

		canvas {
			max-width: 90%;
			margin: 0 auto;
		}
	}
</style>
