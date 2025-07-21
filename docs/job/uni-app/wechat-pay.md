# 支付

## 微信支付

#### 微信支付基本配置 - wxPayConfig

| 标题      | 详情                                           |
| --------- | ---------------------------------------------- |
| appId     | wx2421b1c4370ecxxx（公众号ID，由商户传入）     |
| timeStamp | 1395712654（时间戳，自1970年以来的秒数 ）      |
| nonceStr  | e61463f8efa94090b1f366cccfbbb444（随机串）     |
| package   | prepay_id=wx21201855730335ac86f8c43d1889123400 |
| signType  | RSA（微信签名方式）                            |
| paySign   | oR9d8PuhnIc+YZ8cBHFCwfgp...（微信签名）        |

[H5微信配置](https://pay.weixin.qq.com/wiki/doc/apiv3/open/pay/chapter2_3.shtml)

- getWxPayApi 后端获取微信接口 Api
- orderId 为后端根据订单 ID 返回 微信支付配置
- wxPayConfig 微信支付配置

```javascript
// 选择微信支付
onWxPay() {
    getWxPayApi({orderId})
        .then((res) => {
        if (res.status == "success") {
            // 微信支付配置
            this.wxPayConfig = res.data;
            this.wxPay();
        }
    }).catch((res) => {
        console.log("出错了")
    });
},

// 在微信环境中调用微信支付
wxPay() {
    if (typeof WeixinJSBridge === "undefined") {
        if (document.addEventListener) {
             document.addEventListener(
                "WeixinJSBridgeReady",
                this.onBridgeReady,
                false
            );
        } else if (document.attachEvent) {
            document.attachEvent("WeixinJSBridgeReady", this.onBridgeReady);
            document.attachEvent("onWeixinJSBridgeReady", this.onBridgeReady);
        }
    } else {
        this.onBridgeReady();
    }
},


// 使用微信内置Api调用微信支付
onBridgeReady() {
    let _this = this;
    WeixinJSBridge.invoke(
        "getBrandWCPayRequest",
        this.wxPayConfig,
        function (res) {
            // 由于前端交互复杂，get_brand_wcpay_request:cancel
            //或者get_brand_wcpay_request:fail
            // 可以统一处理为用户遇到错误或者主动放弃，不必细化区分
            if (res.err_msg === "get_brand_wcpay_request:ok") {
                 //支付成功
                console.log("续费成功==>其他操作")
            } else {
                _this.$toast.fail("支付取消");
            }
        }
    );
},
```

## 相关链接

[微信支付官网](https://pay.weixin.qq.com/)
