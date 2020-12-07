<template>
	<view class="container">
		<v-tabs v-model="current" :tabs="category" @change="changeTab" class="tab"></v-tabs>
		<view class="coupon" ref="coupon">
			<view class="item" v-for="(v, i) in projectList" @click="toProject(i)" :key="i">
				<view class="top">
					<view class="left">
						<view class="content">
							<image :src="v.logo" class="icon" mode="widthFix" />
							<view class="name">{{ v.title }}</view>
						</view>
						<view class="text">{{ v.tag ? v.tag : '天天可领' }}</view>
					</view>
					<view class="right">免费领取</view>
				</view>
				<view class="bottom"><image :src="v.image" mode="widthFix" /></view>
			</view>
		</view>
	</view>
</template>

<script>
import CONFIG from '../../common/config.js';
export default {
	data() {
		return {
			current: 0,
			category: [],
			projectList: []
		};
	},
	onLoad(e) {
		//#ifdef H5
		let tabId = this.$route.query.tabId ? parseInt(this.$route.query.tabId) : 0;
		//#endif
		//#ifdef MP-WEIXIN
		let tabId = e.tabId ? parseInt(e.tabId) : 0;
		//#endif
		for (let i in this.tabs) {
			if (tabId == this.tabs[i].tabId) {
				this.current = parseInt(i);
			}
		}
		this.changeTab(this.current);
	},
	onShareAppMessage(res) {
		return {
			title: '美团饿了么大额红包，每日可领',
			path: '/pages/index/index'
		};
	},
	methods: {
		changeTab(index) {
			console.log('当前选中的项：' + index);
			this.couponList = [];
			uni.showLoading({
				title: '获取优惠中'
			});
			uni.request({
				url: `${CONFIG.baseRequestUrl}/api/home/project`,
				data: {
					category_id: index,
					mobile: CONFIG.phone,
					page_size: '1'
				},
				header: {},
				success: res => {
					console.log(res.data);
					let data = res.data.data;
					this.category = data.category;
					let projects = data.project ? data.project.list : [];
					let newProjects = [];
					//#ifdef MP-WEIXIN
					for (let item in projects) {
						if (projects[item].wechat_path) {
							newProjects.push(projects[item]);
						}
					}
					//#endif
					//#ifdef H5
					for (let item in projects) {
						if (projects[item].h5_link) {
							newProjects.push(projects[item]);
						}
					}
					//#endif
					this.projectList = newProjects;
				}
			});
			//#ifdef H5
			this.$nextTick(() => {
				this.$refs.coupon.scrollTop = 0;
			});
			//#endif
			setTimeout(() => {
				uni.hideLoading();
			}, 500);
		},
		toProject(i) {
			console.log(this.projectList[i]);
			//h5
			//#ifdef H5
			window.location.href = this.projectList[i].h5_link;
			//#endif
			//微信小程序
			//#ifdef MP-WEIXIN

			wx.navigateToMiniProgram({
				appId: this.projectList[i].app_id,
				path: this.projectList[i].wechat_path,
				success(res) {
					// 打开成功
				}
			});
			//#endif
		}
	}
};
</script>

<style lang="scss">
page {
	background-color: #f8f8f8;
}
.container {
	font-size: 14px;
	line-height: 24px;
	position: relative;
	.tab {
		position: fixed;
		top: var(--window-top);
		left: 0;
		z-index: 9999;
	}
	.coupon {
		padding-top: 200upx;
		.item {
			background-color: #ffffff;
			margin: 30upx;
			border-radius: 10upx;
			padding: 0 30upx 30upx 30upx;
			.top {
				height: 116upx;
				display: flex;
				align-items: center;
				justify-content: space-between;
				.left {
					height: 116upx;
					display: flex;
					align-items: center;
					justify-content: space-between;
					.content {
						width: 100%;
					}
					.icon {
						display: inline-block;
						vertical-align: bottom;
						width: 52upx;
						height: auto;
					}
					.name {
						text-align: left;
						display: inline-block;
						vertical-align: bottom;
						font-size: 34upx;
						color: #000;
						line-height: 50upx;
						font-weight: bold;
						margin-left: 15upx;
					}
					.text {
						width: 150upx;
						height: 38upx;
						line-height: 38upx;
						text-align: center;
						font-size: 24upx;
						color: #61300e;
						background: linear-gradient(90deg, #f9db8d, #f8d98a);
						border-radius: 6upx;
					}
				}

				.right {
					width: 170upx;
					height: 60upx;
					border-radius: 30upx;
					background: linear-gradient(90deg, #ec6f43, #ea4a36);
					color: #fff;
					font-size: 28upx;
					line-height: 60upx;
					text-align: center;
				}
			}

			.bottom {
				height: auto;
				width: 100%;
				image {
					display: block;
					width: 100%;
					height: auto;
				}
			}
		}
	}
}
</style>
