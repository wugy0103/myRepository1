<template>
  <input type="hidden" id="shareData" :value="JSON.stringify(inputShareData)">
</template>

<script>
import { DxHybrid } from '../../utils/hybrid'
import axios from 'axios'
import ipConfig from '../../config/ipConfig'
import { topicApis } from '../../config/apis'

export default {
  components: {
    axios
  },
  props: {
    // 分享标题
    title: {
      type: String,
      default: '健康商城-正品保证，全场包邮！'
    },
    // 分享描述
    desc: {
      type: String,
      default: '健康猫运动商城-涵盖运动鞋服、营养保健、运动数码等优选正品。'
    },
    // 分享图标
    imgUrl: {
      type: String,
      default: 'https://api.healthmall.cn/logo.jpg'
    },
    // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
    link: {
      type: String,
      default: 'http://m.daxmall.com'
    },
    // 长图分享链接
    longPageUrl: {
      type: String,
      default: ''
    },
    sharePic: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      // 兼容旧分享按钮
      inputShareData: {
        'title': this.title,
        'content': this.desc,
        'img_url': this.imgUrl,
        'http_url': this.link,
        'topic': '#健康商城#',
        'shareFrom': '（分享自@健康猫APP）'
      },
      // 微信分享参数
      wechatShareData: {
        title: this.title,
        desc: this.desc,
        link: this.link,
        imgUrl: this.imgUrl
      },
      QRCodeBase64: ''
    }
  },
  computed: {
    // app分享参数
    appShareData () {
      return {
        'title': this.title,
        'des': this.desc,
        'imgUrl': this.imgUrl,
        'url': this.link,
        'channel': 7,
        'ext': {
          'longPageUrl': this.longPageUrl + encodeURIComponent(this.sharePic) + '/' + encodeURIComponent(this.QRCodeBase64)
        }
      }
    }
  },
  watch: {
    QRCodeBase64 (newval, oldval) {
      this.setAppRightBtn()
      this.appShareInit(this.appShareData)
    }
  },
  methods: {
    getWechatConfig: function (callback) {
      return axios.get(ipConfig.apiBaseUrl + topicApis.wechatUrl, {
        params: { url: location.href }
      }).then((res) => {
        if (callback) callback(res.data)
        return res.data
      }, (rej) => {
        console.log(ipConfig.apiBaseUrl + topicApis.wechatUrl, rej)
        return rej
      })
    },
    wechatInit: function (shareData) {
      return this.getWechatConfig(function (config) {
        let paramObj = {
          appId: config.appId,
          timestamp: config.timestamp,
          nonceStr: config.noncestr,
          signature: config.signature,
          jsApiList: ['checkJsApi',
            'onMenuShareTimeline',
            'onMenuShareAppMessage',
            'onMenuShareQQ',
            'onMenuShareWeibo',
            'hideMenuItems',
            'chooseImage'
          ]
        }
        if (window.wx) {
          let wx = window.wx
          wx.config(paramObj)
          wx.ready(function () {
            wx.onMenuShareTimeline(shareData)
            wx.onMenuShareAppMessage(shareData)
          })
        }
      })
    },
    // 控制app右上角按钮分享图标
    setAppRightBtn: function () {
      DxHybrid.H5callApp('setTBRightButton', {
        'text': '分享',
        'iconId': 'icon1006',
        'enable': true,
        'textColor': '#333333'
      })
      window.onunload = function () {
        DxHybrid.H5callApp('setTBRightButton', {
          'text': '',
          'iconId': '0',
          'enable': false,
          'textColor': '#ffffff'
        })
      }
    },
    appShareInit: function (shareData) {
      DxHybrid.APPCallH5('notifyTBRightButtonClicked', function () {
        DxHybrid.H5callApp('mallToShare', shareData, function () { })
      })
    },
    getQRCodeBase64: function (url) {
      let that = this
      DxHybrid.H5callApp('getQrCode', {'content': url}, function (data) {
        that.QRCodeBase64 = data
      })
    },
    getAppVersion: function () {
      let that = this
      DxHybrid.H5callApp('getAppVersion', {}, function (data) {
        if (parseInt(data) < 3010000) {
          // app分享参数
          that.appShareData.channel = 0
          that.appShareData.ext = {}
          that.setAppRightBtn()
          that.appShareInit(that.appShareData)
        } else {
          that.getQRCodeBase64(that.link)
        }
      })
    }
  },
  mounted () {
    this.getAppVersion()
    this.wechatInit(this.wechatShareData)
  }
}
</script>
