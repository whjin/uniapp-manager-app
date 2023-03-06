<script>
import Api from "@/common/api.js";
import Log from "@/common/utils/log.js";

// 指纹控制
const fingerprint = uni.requireNativePlugin("FingerprintModule");
// 音频控制
const harUtils = uni.requireNativePlugin("HarUtils");
// 语音播报
const base = uni.requireNativePlugin("GK-Base");
const sip = uni.requireNativePlugin("Sip");
// 看门狗
const crashHandle = uni.requireNativePlugin("CrashHandle");
const utils = uni.requireNativePlugin("Utils");
// 读卡器 & 屏幕常亮设置
const cardManager = uni.requireNativePlugin("card-module");

export default {
  onLaunch: function () {
    // #ifdef APP-PLUS
    // 系统状态栏显隐
    harUtils.showStatusBar();
    harUtils.hideStatusBar();
    // 初始化音频增益
    harUtils.initAudioGain();
    harUtils.setVolume(0, uni.getStorageSync("mediaDefaultVolume"));
    harUtils.setVolume(3, uni.getStorageSync("mediaDefaultVolume"));
    // 应急报警按键事件
    harUtils.removeKeyCallBack();
    harUtils.addKeyCallBack();
    // 设置全屏
    plus.navigator.setFullscreen(true);
    // 添加week自定义字体规则
    let domModule = weex.requireModule("dom");
    domModule.addRule("fontFace", {
      fontFamily: "uniicons",
      src: "url('./static/uni.ttf')",
    });
    crashHandle.startGather(0); //开启日志，参数无效，默认保存着
    crashHandle.startGuard(10000); //启动守护，0-不轮询
    // 登录SIP服务
    if (!!uni.getStorageSync("baseUrl")) {
      // SIP登录
      this.initSip();
      // 初始化语音播报
      base.speechInit();
      // 修改分机信息
      this.setTerminalInfo();
    }
    // 开启屏幕常亮
    cardManager.keepScreenOn(res => {
      const r = JSON.parse(res);
      Log.writeLog(`${r.msg}，code：${r.code}`, false);
    });
    // #endif
  },
  onError (err) {
    Log.writeLog(`代码错误${err}`, false);
  },
  methods: {
    // 初始化SIP对讲服务
    initSip () {
      if (sip.logout() === 0) {
        console.log("SIP注销成功");
      }
      if (sip.init() === 0) {
        console.log("SIP初始化成功");
        if (utils.isSelfStart() === 0) {
          // 首次自启动
          sip.startEchoCancellerCalibration();
        }
        let sipAccount = uni.getStorageSync("managerInfo").sipAccount;
        let sipPwd = uni.getStorageSync("sipTalkDevicePwd");
        let sipUrl = uni.getStorageSync("sipTalkServerUrl");
        if (sip.login(sipAccount, sipPwd, sipUrl) === 0) {
          console.log("SIP登录成功");
        } else {
          console.log("SIP登录失败");
        }
      } else {
        console.log("SIP初始化失败");
      }
    },
    // 修改分机信息
    async setTerminalInfo () {
      let params = {
        data: {
          id: uni.getStorageSync("managerInfo").id,
          code: uni.getStorageSync("managerInfo").code,
          version: plus.runtime.version,
        }
      };
      let res = await Api.apiCall("post", Api.index.updateTerminal, params);
      if (res.state.code == "200") {
        console.log("修改信息成功");
      }
    },
  },
  globalData: {
    webSocketConnected: false,
    harUtils: harUtils,
    Base: base,
    fingerprint: fingerprint,
    cardManager
  },
};
</script>

<style>
/* #ifndef APP-PLUS-NVUE */
@import './common/css/uni.css';
@import './static/icons/iconfont.css';
@import './common/css/home-img.css';
@import './common/css/header-img.css';
@import './common/css/navbar-img.css';
@import './common/css/police-img.css';
@import './common/css/bgStyle-img.css';
@import './common/css/neilModal-img.css';
@import './common/css/picker-img.css';
@import './common/css/mood-img.css';
@import './common/css/keyboard-img.css';
@import './common/css/input-img.css';
/* #endif*/
</style>
