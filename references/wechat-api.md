# 微信小程序核心 API 参考

## 用户登录

```typescript
// 微信登录流程
wx.login({
  success: (res) => {
    if (res.code) {
      // 发送 code 到后端换取 openid/session
      wx.request({
        url: 'https://your-api.com/login',
        data: { code: res.code }
      })
    }
  }
})

// 获取用户信息
wx.getUserProfile({
  desc: '用于完善用户资料',
  success: (res) => {
    console.log(res.userInfo)
  }
})
```

## 支付功能

```typescript
// 发起支付
wx.requestPayment({
  timeStamp: '',
  nonceStr: '',
  package: '',
  signType: 'RSA',
  paySign: '',
  success: () => {},
  fail: () => {}
})
```

## 订阅消息

```typescript
// 请求订阅
wx.requestSubscribeMessage({
  tmplIds: ['TEMPLATE_ID'],
  success: (res) => {}
})
```

## 云开发

```typescript
// 初始化云开发
wx.cloud.init({
  env: 'your-env-id',
  traceUser: true
})

// 云函数调用
wx.cloud.callFunction({
  name: 'login',
  data: {}
})

// 云数据库
const db = wx.cloud.database()
db.collection('users').add({ data: {} })
```

## 常用 API

| API | 说明 |
|-----|------|
| `wx.navigateTo` | 页面跳转 |
| `wx.switchTab` | 切换 Tab 页面 |
| `wx.showLoading` | 显示加载提示 |
| `wx.showToast` | 显示消息提示 |
| `wx.setStorage` | 本地存储 |
| `wx.getSystemInfo` | 获取系统信息 |
| `wx.chooseImage` | 选择图片 |
| `wx.uploadFile` | 上传文件 |

## 项目配置

### app.json

```json
{
  "pages": ["pages/index/index"],
  "window": {
    "navigationBarTitleText": "我的小程序"
  },
  "tabBar": {
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页"
      }
    ]
  }
}
```

### project.config.json

```json
{
  "appid": "wxXXXXXXXXXXXX",
  "projectname": "my-miniprogram",
  "miniprogramRoot": "./",
  "cloudfunctionRoot": "./cloudfunctions/"
}
```
