# 支付

## 支付宝支付

#### 支付宝支付配置 - alipayConfig

```json
{
    "alipay_trade_wap_pay_response": {
        "code": "10000",
        "msg": "Success",
        "app_id": "2014072300007148",
        "auth_app_id": "2014072300007148",
        "charset": "UTF-8",
        "timestamp": "2016-10-11 17:43:36",
        "out_trade_no": "081622560194853",
        "total_amount": "9.00",
        "trade_no": "2016081621001004400236957647",
        "seller_id": "2088702849871851"
    },
    "sign": "NGfStJf3i3ooWBuCDIQSumOpaGBcQz+aoAqyGh3W6EqA/gmyPYwLJ********",
    "sign_type": "RSA2"
}
```

- order_sn 订单号

```javascript
	// 获取传值
    this.token = this.query.token;
    this.order_sn = this.query.order_sn;

    // 判断环境
    let ua = navigator.userAgent.toLowerCase();
    if (ua.match(/MicroMessenger/i) == "micromessenger") {
      // 在微信中
      this.$toast.clear();
    } else {
      // 不在微信中
      axios
        .create()({
          headers: { Authorization: "Bearer " + this.token },
          url: process.env.VUE_APP_BASE_API + "/estate/alipay",
          method: "post",
          data: {
            order_sn: this.order_sn,
          },
        })
        .then((res) => {
          this.$toast.clear();
          if (res.data.status == "success") {
            let formDiv = document.getElementsByTagName("formdiv");
            while (formDiv.length) {
              document.body.removeChild(formDiv[0]);
            }
            // 获取系统 ios/android
            const u = navigator.userAgent;
            // const isAndroid = u.indexOf("Android") > -1 || u.indexOf("Adr") > -1; //android终端
            const isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端

            const div = document.createElement("formdiv");
            div.innerHTML = res.data.data;
            document.body.appendChild(div);

            //如果是ios，不新开页面直接跳转
            if (isiOS) {
              document.forms[0].submit();
            } else {
              // document.forms[0].setAttribute("target", "_blank");
              document.forms[0].submit();
            }
          } else {
            this.$toast.fail(res.data.message || "支付失败！");
          }
        })
        .catch((res) => {
          this.$toast.clear();
          this.$toast.fail("出错了，请稍后重试!");
        });
    }


    // 支付状态轮询
    alipayPoll() {
      let _this = this;
      this.timer = setInterval(function () {
        _this.second--;

        // 订单类型判断
        if (_this.type) {
          _this.second % 2 === 0 &&
            axios
              .create()({
                url: `${process.env.VUE_APP_BASE_API}/estate/alipay/backinfo?order_sn=${_this.order_sn}`,
                method: "get",
                headers: { Authorization: "Bearer " + _this.token },
              })
              .then((res) => {
                if (res.data.status == "success" && res.data.data) {
                  //支付成功
                  clearInterval(_this.timer);
                  _this.$toast.success("支付成功");
                  if (_this.type === "prestore") {
                    _this.$router.replace({
                      path: "/estate/prestore/success",
                      query: {
                        order_sn: _this.order_sn,
                      },
                    });
                  } else if (_this.type === "payment") {
                    _this.$router.replace({
                      path: "/estate/payment/success",
                      query: {
                        order_sn: _this.order_sn,
                      },
                    });
                  }
                }
              });
        } else {
          clearInterval(_this.timer);
        }

        if (_this.second <= 0) {
          clearInterval(_this.timer);
          _this.$toast("获取支付结果超时！");
        }
      }, 1000);
    },
        
    // 轮询结果
    this.alipayPoll();
```

## 相关链接

[支付宝支付官网](https://opendocs.alipay.com)
