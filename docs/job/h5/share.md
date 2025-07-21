## 微信公众号分享（H5）版本

##### **后端：**

###### 接口（getWechatConfig）应该返回

```javascript
{
    "debug": false,
    "beta": false,
    "appId": "wx1df7b21d254437a7",
    "nonceStr": "aHRVgFZh3t",
    "timestamp": 1749019018,
    "url": "http://172.16.1.29/pages/goods_details/index?id=8",
    "signature": "b9d36e3a0bcd4a0e999ca777e60edcbaa3525d83",
    "jsApiList": [
        "openAddress",
        "updateTimelineShareData",
        "updateAppMessageShareData",
        "onMenuShareTimeline",
        "onMenuShareAppMessage",
        "onMenuShareQQ",
        "onMenuShareWeibo",
        "onMenuShareQZone",
        "startRecord",
        "stopRecord",
        "onVoiceRecordEnd",
        "playVoice",
        "pauseVoice",
        "stopVoice",
        "onVoicePlayEnd",
        "uploadVoice",
        "downloadVoice",
        "chooseImage",
        "previewImage",
        "uploadImage",
        "downloadImage",
        "translateVoice",
        "getNetworkType",
        "openLocation",
        "getLocation",
        "hideOptionMenu",
        "showOptionMenu",
        "hideMenuItems",
        "showMenuItems",
        "hideAllNonBaseMenuItem",
        "showAllNonBaseMenuItem",
        "closeWindow",
        "scanQRCode",
        "chooseWXPay",
        "openProductSpecificView",
        "addCard",
        "chooseCard",
        "openCard"
    ]
}
```



##### 前端 

###### 安装

```javascript
npm i weixin-js-sdk -S
```

###### 导入

```javascript
import wx from "weixin-js-sdk";
```

```javascript

mounted() {
    // #ifdef H5
    localStorage.setItem("location.href", location.href);
    getWechatConfig({ url: this.getSignUrl() }).then((res) => {
      if (res.status == 200) {
        res.data.timestamp = parseInt(res.data.timestamp);
        res.data.url = window.location.href;
        res.data.jsApiList = ["scanQRCode", ...res.data.jsApiList];
        wx.config(res.data);
        localStorage.setItem("wx.config", res.data);
      }
    });
    // #endif
  },

methods:{
    getSignUrl() {
      let signLink = "";
      const ua = navigator.userAgent.toLowerCase();
      if (/iphone|ipad|ipod/.test(ua)) {
        signLink = localStorage.getItem("location.href");
        if (!signLink) signLink = location.href;
      } else {
        signLink = location.href;
      }
      return signLink;
    },
}
```

###### 页面分享代码

```javascript
    // 全局匹配所有 http:// 前缀（不区分大小写）
    replaceAllHttpToHttps(url) {
      return url.replace(/http:\/\//gi, "https://");
    },
    share() {
      const that = this;
      const time = Date.parse(new Date()) / 1000;
      console.log("link", window.location.href + `&t=${time}`);

      wx.ready(function () {
        const shareData = {
          title: "标题", //标题
          desc:  "描述", //描述
          link: window.location.href + `&t=${time}`, //域名必须JS安全域名
          imgUrl:
            that.replaceAllHttpToHttps(that.storeInfo.image) + `?t=${time}` ||
            "", //封面图片路径
        };
        wx.updateAppMessageShareData(shareData); //1.4 分享到朋友
        wx.updateTimelineShareData(shareData); //1.4分享到朋友圈
      });
    },
```

