<template>
	<view class="content">
		<view class="btn-list">
			<button type="primary" @click="add">新增一条数据</button>
			<button type="primary" @click="remove">删除一条数据</button>
			<button type="primary" @click="update">修改数据</button>
			<button type="primary" @click="get">查询前10条数据</button>
			<button type="primary" @click="useCommon">使用公用模块</button>
			<button type="primary" @click="upload">上传文件</button>
		</view>
		<u-tabbar v-model="current" :list="list" :mid-button="true"></u-tabbar>
	</view>
</template>

<script>
export default {
	data() {
		return {
			list: [
				{
					iconPath: 'home',
					selectedIconPath: 'home-fill',
					text: '首页',
					count: 2,
					isDot: true,
					customIcon: false
				},
				// {
				// 	iconPath: 'photo',
				// 	selectedIconPath: 'photo-fill',
				// 	text: '放映',
				// 	customIcon: false
				// },
				{
					iconPath: 'https://cdn.uviewui.com/uview/common/min_button.png',
					selectedIconPath: 'https://cdn.uviewui.com/uview/common/min_button_select.png',
					text: '132',
					midButton: true,
					customIcon: false
				},
				// {
				// 	iconPath: 'play-right',
				// 	selectedIconPath: 'play-right-fill',
				// 	text: '直播',
				// 	customIcon: false
				// },
				{
					iconPath: 'account',
					selectedIconPath: 'account-fill',
					text: '我的',
					count: 23,
					isDot: false,
					customIcon: false
				}
			],
			current: 0
		};
	},
	methods: {
		add() {
			uni.showLoading({
				title: '处理中...'
			});
			uniCloud
				.callFunction({
					name: 'add',
					data: {
						name: 'DCloud',
						subType: 'uniCloud',
						createTime: Date.now()
					}
				})
				.then(res => {
					uni.hideLoading();
					uni.showModal({
						content: `成功添加一条数据，文档id为：${res.result.id}`,
						showCancel: false
					});
					console.log(res);
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `添加数据失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.error(err);
				});
		},
		remove() {
			uni.showLoading({
				title: '处理中...'
			});
			uniCloud
				.callFunction({
					name: 'remove'
				})
				.then(res => {
					uni.hideLoading();
					uni.showModal({
						content: res.result.msg,
						showCancel: false
					});
					console.log(res);
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `删除失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.error(err);
				});
		},
		update() {
			uni.showLoading({
				title: '处理中...'
			});
			uniCloud
				.callFunction({
					name: 'update',
					data: {
						name: 'DCloud',
						subType: 'html 5+',
						createTime: Date.now()
					}
				})
				.then(res => {
					uni.hideLoading();
					uni.showModal({
						content: res.result.msg,
						showCancel: false
					});
					console.log(res);
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `更新操作执行失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.error(err);
				});
		},
		get() {
			uni.showLoading({
				title: '处理中...'
			});
			uniCloud
				.callFunction({
					name: 'get'
				})
				.then(res => {
					uni.hideLoading();
					uni.showModal({
						content: `查询成功，获取数据列表为：${JSON.stringify(res.result.data)}`,
						showCancel: false
					});
					console.log(res);
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `查询失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.error(err);
				});
		},
		useCommon() {
			console.log('请确保自己已经阅读并按照公用模块文档操作 https://uniapp.dcloud.io/uniCloud/cf-common');
			uniCloud
				.callFunction({
					name: 'use-common'
				})
				.then(res => {
					uni.hideLoading();
					uni.showModal({
						content: '云函数use-common返回结果：' + JSON.stringify(res.result),
						showCancel: false
					});
					console.log(res);
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `云函数use-common执行失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.error(err);
				});
		},
		upload() {
			new Promise((resolve, reject) => {
				uni.chooseImage({
					count: 1,
					success: res => {
						const path = res.tempFilePaths[0];
						let ext;
						// #ifdef H5
						ext = res.tempFiles[0].name.split('.').pop();
						const options = {
							filePath: path,
							cloudPath: Date.now() + '.' + ext
						};
						resolve(options);
						// #endif
						// #ifndef H5
						uni.getImageInfo({
							src: path,
							success(info) {
								const options = {
									filePath: path,
									cloudPath: Date.now() + '.' + info.type.toLowerCase()
								};
								resolve(options);
							},
							fail(err) {
								reject(new Error(err.errMsg || '未能获取图片类型'));
							}
						});
						// #endif
					},
					fail: () => {
						reject(new Error('Fail_Cancel'));
					}
				});
			})
				.then(options => {
					uni.showLoading({
						title: '文件上传中...'
					});
					return uniCloud.uploadFile({
						...options,
						onUploadProgress(e) {
							console.log(e);
						}
					});
				})
				.then(res => {
					uni.hideLoading();
					console.log(res);
					uni.showModal({
						content: '图片上传成功，fileId为：' + res.fileID,
						showCancel: false
					});
				})
				.catch(err => {
					uni.hideLoading();
					console.log(err);
					if (err.message !== 'Fail_Cancel') {
						uni.showModal({
							content: `图片上传失败，错误信息为：${err.message}`,
							showCancel: false
						});
					}
				});
		}
	}
};
</script>

<style>
.content {
	padding-bottom: 30px;
}

.btn-list {
	padding: 0px 30px;
}

.btn-list button {
	margin-bottom: 20px;
}

.upload-preview {
	width: 100%;
}
</style>
