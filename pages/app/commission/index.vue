<!-- 佣金中心 -->
<template>
	<view style="width:100%;height: 100%;">
		<view class="commission-wrap" :class="{ blur: !hasAuth }">
			<cu-custom isBack></cu-custom>
			<!-- 用户资料 -->
			<view class="user-card">
				<view class="card-top x-f">
					<view class="head-img-box"><image class="head-img" :src="userInfo.avatar" mode="widthFix"></image></view>
					<view class="y-start">
						<view class="user-info-box x-f">
							<view class="user-name">{{ userInfo.nickname }}</view>
							<view class="grade-tag tag-box x-f" v-if="commissionLv">
								<image class="tag-img" :src="commissionLv.image" mode=""></image>
								<text class="tag-title">{{ commissionLv.name }}</text>
							</view>
						</view>
						<view class="progress-box">
							<view class="cu-progress round sm">
								<view class="progress--ing" :style="[{ width: '50%' }]"></view>
								<view class="round-wrap"><view class="round-inner"></view></view>
							</view>
							<view class="progress-tip">距离下次升级还差100财富值</view>
						</view>
					</view>
				</view>
				<view class="card-bottom x-f" v-if="commissionWallet">
					<view class="flex-sub y-start">
						<view class="item-title">总收益</view>
						<view class="item-detail">{{ showMoney ? commissionWallet.total_income || '0.00' : '***' }}</view>
					</view>
					<view class="flex-sub  y-start">
						<view class="item-title">待入账佣金</view>
						<view class="item-detail">{{ showMoney ? commissionWallet.delay_money || '0.00' : '***' }}</view>
					</view>
					<view class="y-f">
						<button class="cu-btn log-btn" @tap="jump('/pages/app/commission/commission-log')">明细</button>
						<button class="cu-btn look-btn" @tap="onEye">
							<text v-if="showMoney" class="cuIcon-attentionfill"></text>
							<text v-else class="cuIcon-attentionforbidfill"></text>
						</button>
					</view>
				</view>
			</view>
			<!-- 滚动明细 -->
			<view class="commission-log">
				<scroll-view class="log-scroll" scroll-y="true">
					<view class="log-item-box" v-for="item in 12" :key="item">
						<view class="log-item x-f">
							<image class="log-img" src="http://shopro.7wpp.com/imgs/app_icon/icon.png" mode=""></image>
							<view class="log-text">139****2561 刚购买了家用小冰箱，还未付款</view>
						</view>
					</view>
				</scroll-view>
			</view>
			<!-- 功能菜单 -->
			<image class="commission-bottom-bg" src="http://shopro.7wpp.com/imgs/commission/commission_bottom.png" mode="widthFix"></image>
			<view class="menu-box flex">
				<view class="menu-item y-f" v-for="(menu, index) in menuList" v-if="!menu.isAgentFrom" :key="index" @tap="jump(menu.path)">
					<image class="item-img" :src="menu.img" mode=""></image>
					<view class="item-title">{{ menu.title }}</view>
				</view>
			</view>
			<!-- 登录提示 -->
			<shopro-login-modal></shopro-login-modal>
		</view>
		<!-- 佣金中心权限验证 -->
		<view class="auth-box cu-modal" :class="{ show: !hasAuth }">
			<view class="notice-box cu-dialog">
				<view class="img-wrap"><image class="notice-img" :src="authNotice.img" mode=""></image></view>
				<view class="notice-title">{{ authNotice.title }}</view>
				<view class="notice-detail">{{ authNotice.detail }}</view>
				<button class="cu-btn notice-btn" @tap="onAuthBtn(authNotice.btnPath)">{{ authNotice.btnText }}</button>
				<button class="cu-btn back-btn" @tap="onBack">返回</button>
			</view>
		</view>
	</view>
</template>

<script>
import { mapMutations, mapActions, mapState } from 'vuex';
export default {
	components: {},
	data() {
		return {
			showMoney: true, //是否显示金额
			hasAuth: true, //是否有权限
			commissionLv: null, //分销商等级
			commissionWallet: null, //分销商钱包
			agentFrom: null, //是否显示我的资料
			authNotice: {
				//权限提示内容
				img: 'http://shopro.7wpp.com/imgs/commission/auth_check.png',
				title: '正在审核中！',
				detail: '请耐心等候',
				btnText: '知道了'
			},
			menuList: [
				//menu
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon1.png',
					title: '我的团队',
					path: '/pages/app/commission/team'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon2.png',
					title: '佣金明细',
					path: '/pages/app/commission/commission-log'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon3.png',
					title: '分销订单',
					path: '/pages/app/commission/order'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon4.png',
					title: '推广商品',
					path: '/pages/app/commission/goods'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon5.png',
					title: '我的资料',
					path: '/pages/app/commission/apply',
					isAgentFrom: true
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon6.png',
					title: '分销排行',
					path: '/pages/app/commission/rankings'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon7.png',
					title: '邀请海报',
					path: '/pages/public/poster/index?posterType=user'
				},
				{
					img: 'http://shopro.7wpp.com/imgs/commission/commission_icon8.png',
					title: '分享记录',
					path: '/pages/app/commission/share-log'
				}
			]
		};
	},
	computed: {
		...mapState({
			userInfo: state => state.user.userInfo
		})
	},
	onShow() {
		this.getStatus();
	},
	onPullDownRefresh() {
		this.getStatus();
	},
	methods: {
		...mapActions(['getAgent']),
		// 跳转
		jump(path, query) {
			this.$tools.routerTo(path, query);
		},

		// 是否显示金额
		onEye() {
			this.showMoney = !this.showMoney;
		},

		// 身份认证
		getStatus() {
			let that = this;
			this.getAgent().then(res => {
				uni.stopPullDownRefresh();
				if (res.code === 1) {
					that.authStatus(res.data);
					that.commissionWallet = res.data.data;
					that.commissionLv = res.data.data.agent_level;
					that.menuList.map(item => {
						if (item.title === '我的资料') {
							item.isAgentFrom = !res.data.agent_form;
						}
					});
				}
			});
		},
		// 状态鉴权
		authStatus(data) {
			switch (data.status) {
				case 'forbidden':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_stop.png',
						title: '抱歉！你的账户已被禁用',
						detail: data.msg,
						btnText: '联系客服',
						btnPath: '/pages/public/kefu/index'
					};
					break;
				case 'pending':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_stop.png',
						title: '正在审核中！',
						detail: data.msg,
						btnText: '知道了',
						btnPath: ''
					};
					break;
				case 'null':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_stop.png',
						title: '抱歉！你的账户已被禁用',
						detail: data.msg,
						btnText: '联系客服',
						btnPath: '/pages/public/kefu/index'
					};
					break;
				case 'needinfo':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_perfect.png',
						title: '待完善信息',
						detail: data.msg,
						btnText: '去完善',
						btnPath: '/pages/app/commission/apply'
					};
					break;
				case 'reject':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_reject.png',
						title: '申请驳回',
						detail: data.msg,
						btnText: '修改资料',
						btnPath: '/pages/app/commission/apply'
					};
					break;
				case 'close':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_close.png',
						title: '分销中心已关闭',
						detail: data.msg,
						btnText: '我知道了',
						btnPath: ''
					};
					break;
				case 'freeze':
					this.hasAuth = false;
					this.authNotice = {
						img: 'http://shopro.7wpp.com/imgs/commission/auth_freeze.png',
						title: '抱歉！你的账户已被冻结',
						detail: data.msg,
						btnText: '联系客服',
						btnPath: '/pages/public/kefu/index'
					};
					break;
				default:
					this.hasAuth = true;
			}
		},

		// 权限
		onAuthBtn(path) {
			if (path) {
				this.$Router.push({
					path: path
				});
			} else {
				uni.navigateBack({
					delta: 1
				});
			}
		},
		onBack() {
			uni.navigateBack({
				delta: 1
			});
		}
	}
};
</script>

<style lang="scss">
.commission-wrap {
	background: url('http://shopro.7wpp.com/imgs/commission/commission_bg1.jpg') no-repeat;
	background-size: cover;
	height: 100%;
	width: 100%;
	background-position: center center;
	transition: all 0.3s ease-in-out 0s;
	overflow: hidden;
	/deep/ .cu-back {
		color: #fff;
		font-size: 38rpx;
	}
}
// 佣金中心权限验证蒙层
.blur {
	filter: blur(20rpx);
	transition: all 0.3s ease-in-out 0s;
}

.auth-box {
	width: 100%;
	height: 100%;
	overflow: hidden;
	position: fixed;
	z-index: 99;
	background: none;
	.notice-box {
		position: fixed;
		z-index: 1111;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: #fff;
		top: 50%;
		left: 50%;
		width: 612rpx;
		min-height: 658rpx;
		background: #ffffff;
		padding: 30rpx;
		border-radius: 20rpx;
		transform: translate(-50%, -50%);
		.img-wrap {
			margin-bottom: 50rpx;
			.notice-img {
				width: 180rpx;
				height: 170rpx;
			}
		}
		.notice-title {
			font-size: 35rpx;
			font-weight: bold;
			color: #46351b;
			margin-bottom: 28rpx;
		}
		.notice-detail {
			font-size: 28rpx;
			font-weight: 400;
			color: #999999;
			line-height: 36rpx;
			margin-bottom: 50rpx;
		}
		.notice-btn {
			width: 492rpx;
			height: 70rpx;
			background: linear-gradient(-90deg, #a36fff, #5336ff);
			box-shadow: 0px 7rpx 11rpx 2rpx rgba(124, 103, 214, 0.34);
			border-radius: 35rpx;
			font-size: 28rpx;
			font-weight: 500;
			color: #ffffff;
			margin-bottom: 10rpx;
		}
		.back-btn {
			width: 492rpx;
			height: 70rpx;
			font-size: 28rpx;
			font-weight: 500;
			color: #c5b8fb;
			background: none;
		}
	}
}
.show-auth-box {
	opacity: 1 !important;
	transform: scale(1) !important;
	transition: all 0.3s ease-in-out 0s;
}
// 用户资料卡片
.user-card {
	width: 690rpx;
	min-height: 350rpx;
	border-radius: 14rpx;
	margin: 30rpx;
	background: url('http://shopro.7wpp.com/imgs/commission/commission_card_bg.png') no-repeat;
	background-size: 100% 100%;
	padding-top: 10rpx;
	.card-top {
		margin: 40rpx;
	}
	.head-img-box {
		margin-right: 26rpx;
		width: 76rpx;
		height: 76rpx;
		border-radius: 50px;
		position: relative;
		background: #fff;
		padding: 10rpx;
		background-clip: padding-box;
		.head-img {
			width: 66rpx;
			height: 66rpx;
			border-radius: 50%;
			position: absolute;
			top: 50%;
			left: 50%;
			transform: translate(-50%, -50%);
		}
	}
	.user-info-box {
		.user-name {
			font-size: 26rpx;
			font-weight: 500;
			color: #ffffff;
			line-height: 30rpx;
		}
		.tag-box {
			background: rgba(0, 0, 0, 0.2);
			border-radius: 21rpx;
			line-height: 38rpx;
			padding-right: 10rpx;
			margin-left: 10rpx;

			.tag-img {
				width: 36rpx;
				height: 36rpx;
				margin-right: 6rpx;
				border-radius: 50%;
			}

			.tag-title {
				font-size: 20rpx;
				font-family: PingFang SC;
				font-weight: 500;
				color: rgba(255, 255, 255, 1);
				line-height: 20rpx;
			}
		}
	}
	.progress-box {
		.progress-tip {
			font-size: 16rpx;
			font-weight: 500;
			color: #c7b2ff;
			line-height: 30rpx;
			margin-top: 10rpx;
		}
		.cu-progress {
			width: 150rpx;
			height: 10rpx;
			background: #503bae;
			overflow: visible;
			.progress--ing {
				background: linear-gradient(180deg, #c6a6ff 0%, #7430ee 100%);
				border-radius: 10rpx;
			}
			.round-wrap {
				width: 30rpx;
				height: 30rpx;
				border: 2rpx solid #5c3cff;
				background: linear-gradient(0deg, #a36fff 0%, #846fff 100%);
				border-radius: 50%;
				position: relative;
				left: -15rpx;
				.round-inner {
					width: 20rpx;
					height: 20rpx;
					border: 2rpx solid #5c3cff;
					background: linear-gradient(0deg, #a36fff 0%, #846fff 100%);
					border-radius: 50%;
					position: absolute;
					top: 50%;
					left: 50%;
					transform: translate(-50%, -50%);
				}
			}
		}
	}
	.card-bottom {
		margin: 0 40rpx 40rpx;
		.item-title {
			font-size: 24rpx;
			font-family: PingFang SC;
			font-weight: 400;
			color: #ffffff;
			line-height: 30rpx;
		}
		.item-detail {
			font-size: 40rpx;
			font-family: DIN;
			font-weight: 500;
			color: #fefefe;
			line-height: 30rpx;
			margin-top: 30rpx;
		}
		.cu-btn {
			padding: 0;
			background: none;
		}
		.log-btn {
			width: 83rpx;
			height: 41rpx;
			border: 1rpx solid rgba(#ffffff, 0.33);
			border-radius: 21rpx;
			font-size: 22rpx;
			font-weight: 400;
			color: #ffffff;
		}
		.look-btn {
			color: #fff;
			font-size: 40rpx;
			.cuIcon-attentionfill,
			.cuIcon-attentionforbidfill {
				line-height: 50rpx;
				margin-top: 20rpx;
			}
			.cuIcon-attentionfill {
				line-height: 50rpx;
				margin-top: 20rpx;
			}
		}
	}
}
// 滚动明细
.commission-log {
	min-height: 450rpx;
	padding: 0 30rpx;
	margin-top: 60rpx;
	.log-scroll {
		height: 300rpx;
		.log-item-box {
			position: relative;
			height: 78rpx;
		}
		.log-item {
			position: absolute;
			height: 48rpx;
			background: rgba(#5e49c3, 0.4);
			border-radius: 24rpx;
			padding-left: 6rpx;
			padding-right: 20rpx;
			margin-bottom: 20rpx;
			.log-img {
				width: 40rpx;
				height: 40rpx;
				border-radius: 50%;
				margin-right: 10rpx;
			}
			.log-text {
				font-size: 22rpx;
				font-weight: 500;
				color: #fefefe;
			}
		}
	}
}
// 功能菜单
.commission-bottom-bg {
	position: fixed;
	width: 100%;
	bottom: 0;
	z-index: 6;
}
.menu-box {
	flex-wrap: wrap;
	margin: 30rpx;
	position: fixed;
	width: 750rpx;
	bottom: 0;
	z-index: 10;
	.menu-item {
		width: (690rpx/4);
		margin-bottom: 50rpx;
		.item-img {
			width: 68rpx;
			height: 74rpx;
		}
		.item-title {
			font-size: 24rpx;
			font-weight: 500;
			color: #4e7fa0;
			line-height: 30rpx;
			margin-top: 16rpx;
			text-shadow: 0 0 10rpx #fff, 0 0 10rpx #fff, 0 0 10rpx #fff, 0 0 10rpx #fff;
		}
	}
}
</style>
