# TRTC on Electron


###  一、环境搭建

#### 1、windows 
前往[下载地址](https://nodejs.org/en/download/)，选择windows 32-bit版本，安装node环境。
注意: trtc electron sdk暂只支持win32版本，安装node 请选用win32版本

#### 2、mac
1、安装node：brew install node


## 二、跑通Demo
#### 1、下载地址
https://github.com/tencentyun/TRTCSDK

#### 2、找到Appid和密钥
进入腾讯云实时音视频 [控制台](https://console.cloud.tencent.com/rav) 创建一个新的应用，获得 SDKAppID，SDKAppID 是腾讯云后台用来区分不同实时音视频应用的唯一标识，在第3步中会用到。

![](https://main.qcloudimg.com/raw/b9d211494b6ec8fcea765d1518b228a1.png)

在[快速上手](https://console.cloud.tencent.com/rav/app/1400251387/guide)查看并拷贝加密密钥，点击**查看密钥**按钮，即可看到用于计算 UserSig 的加密密钥，点击“复制密钥”按钮，可以将密钥拷贝到剪贴板中。
![](https://main.qcloudimg.com/raw/6ae2ee8958c5147a591a81136320fe21.png)

#### 3、粘贴密钥到Demo工程的指定文件中

我们在各个平台的 Demo 的源码工程中都提供了一个叫做 “GenerateTestUserSig” 的文件(目录:TRTC_Electron_Demo/js/GenerateTestUserSig.js)，它可以通过 HMAC-SHA256 算法本地计算出 UserSig，用于快速跑通 Demo。您只需要将第1步中获得的 SDKAppID 和第3步中获得的加密密钥拷贝到文件中的指定位置即可，如下所示：

![](https://main.qcloudimg.com/raw/9275a5f99bf00467eac6c34f6ddd3ca5.jpg)

#### 4、开始运行
通过命令行工具，在TRTC_Electron_Demo目录下:
```js
npm install  //下载npm包
npm start    //开始运行
```

## 三、参考文档
https://trtc-1252463788.file.myqcloud.com/electron_sdk/docs/

## 四、TRTC_Electron 使用方法

#### 1、通过npm下载trtc库(确认联网状态)
```
npm install trtc-electron-sdk -S 
```

#### 2、具体使用方法
```js
//1、引入库
const TRTCCloud = require('trtc-electron-sdk');
const {
    TRTCVideoStreamType,
    TRTCAppScene,
    TRTCVideoResolution,
    TRTCVideoResolutionMode,
    TRTCParams
} = require('trtc-electron-sdk/liteav/trtc_define');

//2、构建 TRTCCloud
this.rtcCloud = new TRTCCloud();

//3、注册回调
subscribeEvents = (rtcCloud) => {
    rtcCloud.on('onError', (errcode, errmsg) => {
        console.info('trtc_demo: onError :' + errcode + " msg" + errmsg);
    }); 

    rtcCloud.on('onEnterRoom', (elapsed) => {
        console.info('trtc_demo: onEnterRoom elapsed:' + elapsed);
    });
    rtcCloud.on('onExitRoom', (reason) => {
        console.info('onExitRoom: userenter reason:' + reason);
    });

    // 注册远程视频的可用状态
    rtcCloud.on('onUserVideoAvailable', (uid, available) => {
        console.info('trtc_demo: onUserVideoAvailable uid:' + uid + " available:" + available);
        if (available) {
            let view = this.findVideoView(uid, TRTCVideoStreamType.TRTCVideoStreamTypeBig);
            this.rtcCloud.startRemoteView(uid, view);
        }
        else {
            this.rtcCloud.stopRemoteView(uid);
            this.destroyVideoView(uid, TRTCVideoStreamType.TRTCVideoStreamTypeBig);
        }
    });

    //.....
    //.....
};
subscribeEvents(this.rtcCloud);

//4、进入房间
enterroom () {
    //1. 进房参数
    let param = new TRTCParams();
    param.sdkAppId = sdkInfo.sdkappid;
    param.roomId = parseInt(this.roomId);
    param.userSig = userSig;
    param.userId = this.userId;
    param.privateMapKey = '';
    param.businessInfo = '';
    this.rtcCloud.enterRoom(param, TRTCAppScene.TRTCAppSceneVideoCall);
    //2. 编码参数
    let encparam = new TRTCVideoEncParam();
    encparam.videoResolution = TRTCVideoResolution.TRTCVideoResolution_640_360;
    encparam.resMode = TRTCVideoResolutionMode.TRTCVideoResolutionModeLandscape;
    encparam.videoFps = 15;
    encparam.videoBitrate = 550;
    this.rtcCloud.setVideoEncoderParam(encparam);
    //3. 打开采集和预览本地视频、采集音频
    enableVideoCapture(true);
    enableAudioCapture(true);
},

//5、退出房间
exitroom() {
    this.rtcCloud.exitRoom();
},

//6、开启视频
enableVideoCapture(bEnable) {
    if (bEnable) {
        let view = this.findView("local", TRTCVideoStreamType.TRTCVideoStreamTypeBig);
        this.rtcCloud.startLocalPreview(view);
    }
    else {
        this.rtcCloud.stopLocalPreview();
    }
},

//7、开启音频
enableAudioCapture(bEnable) {
    if (bEnable) {
        this.rtcCloud.startLocalAudio();
    }
    else {
        this.rtcCloud.stopLocalAudio();
    }
},

//8、找个DOM结点，作为视频显示的view
findVideoView(uid, streamtype) {
    let key = uid + String(streamtype);
    var userVideoEl = document.getElementById(key);
    if (!userVideoEl) {
    userVideoEl = document.createElement('div');
    userVideoEl.id = key;
    userVideoEl.classList.add('video_view');
    document.querySelector("#video_wrap").appendChild(userVideoEl);
    }
    return userVideoEl;
},

//9、在视频退出时，清掉一个DOM结点
destroyVideoView(uid, streamtype) {
    let key = uid + String(streamtype);
    var userVideoEl = document.getElementById(key);
    if (userVideoEl) {
    document.querySelector("#video_wrap").removeChild(userVideoEl);
    }
},

```



  