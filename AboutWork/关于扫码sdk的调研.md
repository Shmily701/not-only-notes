
     现状
     目前在使用的华为sdk，支持：一维码二维码，问题：对于倾斜的二维码扫码效果较差

     免费 vs HW 
     zixing  接入后未有明显提升，甚至更差
     zbar    接入后未有明显提升（ 比zixing效果好一些）
     OpenCV  Demo体验效果很好，开源的微信二维码引擎移植的封装库，尝试接入，问题:不支持一维码
     https://github.com/jenly1314/WeChatQRCode [] !!!

     MLKit   Google提供的移动端机器学习库，比华为的sdk好一些，问题：对于倾斜的二维码扫码效果较差
     华为sdk 目前在使用的，支持：一维码二维码，问题：对于倾斜的二维码扫码效果较差
   
     
     收费
     智能扫码  仅demo体验，未有明显提升且收费-不考虑     
     大名 看视频效果很好，但没有demo体验，且收费-不考虑


WeChatQRCode 
1. 引擎：微信二维码引擎，有已编译好的OpenCV；全面的cpu架构so库，
2. 识别功能 wechat-qrcode 
3. 扫码功能 wechat-qrcode-scanning 
4. 相机核心库 MLKit mlkit-camera-core


