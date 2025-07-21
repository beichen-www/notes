# 支付

## 微信And支付宝(封装)

#### 支付宝支付配置 - alipayConfig

- ##### utils文件中  添加 payment.js

```javascript
export const payment = (pay_config, pay_type, success, fail) => {

	// 支付宝支付
	if (pay_type == "alipay") {
		uni.requestPayment({
			"provider": "alipay",
			"orderInfo": pay_config,
			success: function(res) {
				success(res)
			},
			fail: function(err) {
				fail(err)
			},
			complete: function(err) {},
		});
	}
	
	//微信支付 
	if(pay_type == "wxpay"){
		uni.requestPayment({
			"provider": "wxpay",
			"orderInfo": pay_config,
			success(res) {
				success(res)
			},
			fail(err) {
				fail(err)
			}
		})
	}

}

/**
 * @param {String} pay_config 
 * @param {Function} success 
 * @param {Function} fail 
 * @param {Function} complete 
 */
function aliAppPay(pay_config, success, fail, complete) {
	uni.requestPayment({
		"provider": "alipay",
		"orderInfo": pay_config,
		success: function(res) {
			uni.$u.toast(res.message);
		},
		fail: function(err) {
			uni.$u.toast(err.message);
		},
		complete: function(err) {
			complete(err)
		},
	});
	return
}


/*跳到支付成功页*/
function paySuccess(pay_config, self, success) {
	if (success) {
		success(pay_config);
		return;
	}
	gotoSuccess(pay_config);
}

/*跳到支付成功页*/
function gotoSuccess(pay_config) {
	uni.redirectTo({
		url: '/pages/my/order/order-all'
	});
}


/*支付失败跳订单详情*/
function payError(pay_config, fail) {
	if (fail) {
		fail(pay_config);
		return;
	}
	uni.redirectTo({
		url: '/pages/order/order-detail?order_id=' + pay_config.data.order_id
	});
}

```
#### 组件封装

```javascript
<template>
	<view>
		<u-popup :show="payShow" :round="30" mode="bottom">
			<view class="pay-ctn">
				<view class="pay-header">
					<view class="head-tips">
						<text class="pay">{{title}}</text>
						<text class="pay-price">￥{{ amount || "--" }}</text>
					</view>
					<view class="pay-header-close" @tap="close"><text class="ri-close-fill"></text></view>
				</view>
				<view class="pay-list">
					<block v-for="(item, index) in payList" :key="index">
						<view class="pay-list-item" @tap="electPay(item.pay_type)">
							<image class="image-icon" :src="item.icon" mode=""></image>
							<view class="brank">{{ item.name }}</view>
							<text class="ri-arrow-drop-right-line"></text>
						</view>
					</block>
				</view>
			</view>
		</u-popup>

		<!-- 杉德支付-->
		<view v-if="false" ref="payform" v-html="fromData"></view>
	</view>
</template>

<script>
	import {
		postAlipayApi,
	} from "@/api/common/common"
	import {
		payment
	} from '@/utils/payment';
	export default {
		name: "pay-popup",
		props: {
			payShow: {
				type: Boolean,
				default: true
			},
			amount: {
				type: String | Number,
				default: '--'
			},
			title: {
				type: String,
				default: '支付金额：'
			},
			postData: {
				type: String | Object | Array,
				default: ""
			}
		},
		data() {
			return {
				fromData: "",
				// 弹层
				payList: [],
				from: {
					amount: ""
				},
			};
		},

		mounted() {
			this.payList = [{
					icon: "/static/images/common/alipay-icon.png",
					name: "支付宝支付",
					pay_type: "alipay",
				},
				{
					icon: "/static/images/common/wxpay-icon.png",
					name: "微信支付",
					pay_type: "wxpay",
				},
				{
					icon: "/static/images/common/sandpay-icon.png",
					name: "杉德支付",
					pay_type: "sandpay",
				},
			];
		},
		methods: {
            // 选择支付方式
			electPay(pay_type) {
				switch (pay_type) {
					case "sandpay":
						this.postSandpay(this.postData);
						break;
					case "alipay":
						this.postAlipay(this.postData);
						break;
					case "wxpay":
						this.postWxpay(this.postData);
						break;
					default:
						break;
				}

			},

			// 杉德支付
			postSandpay(postData) {
				return
				postAlipayApi(postData)
					.then((res) => {
						this.loadingShow = 0;
						if (res.status == "success") {
							this.fromData = res.data;
							// #ifndef APP-PLUS
							setTimeout(() => {
								this.$refs.payform.$el.children[0].submit();
							}, 0);
							// #endif
							// #ifdef APP-PLUS
							uni.navigateTo({
								url: `/pages/sandpay/sandpay?form=${encodeURIComponent(
			               res.data
			             )}`,
							});
							// #endif
						} else {
							uni.$u.toast(res.message);
						}
					})
					.catch((err) => {
						this.loadingShow = 0;
					});
			},

			// 微信支付
			postWxpay(postData) {
				return
				postAlipayApi(postData)
					.then((res) => {
						if (res.status == "success") {
							let pay_config = res.data
							// 微信支付
							payment(pay_config, 'wxpay')
						} else {
							uni.$u.toast(res.message);
						}
					})
			},

			// 支付宝支付
			postAlipay(data) {
				console.log("data", data);
				postAlipayApi(data)
					.then((res) => {
						if (res.status == "success") {
							let pay_config = res.data
							pay_config && payment(pay_config, 'alipay', (res) => {
								this.$emit('close', false)
								this.$emit('paySuccess', res)
							})
						} else {
							uni.$u.toast(res.message);
							this.$emit('close', false)
							this.$emit('payError', res)
						}
					})
			},

			close() {
				this.$emit('close', false)
			},
		}
	}
</script>

<style lang="scss" scoped>
	.pay-ctn {
		.pay-header {
			position: relative;
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 40rpx;
			border-bottom: 2rpx solid var(--gray-200);
			flex-shrink: 0;
			background-color: #fff;

			.head-tips {
				font-size: 36rpx;
				margin: 0;
				font-weight: 600;
				color: var(--dark);

				.pay {
					color: #000;
				}

				.pay-price {
					color: #ff543e;
				}
			}

			&-close {
				width: 60rpx;
				height: 60rpx;
				display: inline-flex;
				align-items: center;
				justify-content: center;
				font-size: 36rpx;
				font-weight: 600;
				color: var(--dark);
				background-color: #fff;
				border-radius: 50%;
				text-decoration: none !important;
			}
		}

		.pay-list {
			background-color: #fff;
			padding: 40rpx;
			position: relative;
			flex: 1 1 auto;

			&-item {
				height: 80rpx;
				margin-bottom: 24rpx;
				padding: 24rpx 40rpx;
				display: flex;
				align-items: center;
				justify-content: space-between;
				border-radius: 20rpx;
				border: 2rpx solid var(--gray-200);
				overflow: hidden;
				z-index: 1;

				.brank {
					width: 400rpx;
					font-size: 32rpx;
				}

				.image-icon {
					width: 48rpx;
					height: 48rpx;
				}

				text {
					font-size: 48rpx;
					color: var(--secondary);

					.ri-arrow-drop-right-line {
						width: 40rpx;
					}
				}
			}
		}

		.pay-pwd {
			padding: 80rpx 40rpx;
			display: flex;
			justify-content: center;
		}

		.pay-btn {
			display: flex;
			justify-content: center;
			align-items: center;
			margin: 0 20rpx;
			padding-bottom: 100rpx;

			view {
				min-width: 240rpx;
				height: 90rpx;
				padding-left: 40rpx;
				padding-right: 40rpx;
				margin: 0 20rpx;
				font-size: 28rpx;
				font-weight: 500;
				display: inline-flex;
				align-items: center;
				justify-content: center;
				border-radius: 100rpx;
				border: 2rpx solid var(--gray-200);
				box-sizing: border-box;
			}

			&-sure {
				background-color: #395aff;
				color: #fff;
			}
		}
	}
</style>
```

## 相关链接

[支付宝支付官网](https://opendocs.alipay.com)

[微信支付官网](https://pay.weixin.qq.com/)
