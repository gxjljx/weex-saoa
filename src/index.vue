<template>
  <div class="wrapper" v-if="indexShow">
    <x-tabbar :tab-titles="tabTitles" 
        :tab-styles="tabStyles"
        title-type="iconFont"
        @wxcTabBarCurrentTabSelected="wxcTabBarCurrentTabSelected">
    <div class="item-container" :style="contentStyle">
      <message ref="messageCom"></message>
    </div>
    <div class="item-container" :style="contentStyle">
      <appcenter ref="appcenterCom" :theme="centerTheme"></appcenter>
    </div>
    <div class="item-container" :style="contentStyle">
      <maillist ref="maillistCom"></maillist>
    </div>
    <div class="item-container" :style="contentStyle">
      <my ref="myinfo"></my>
    </div>
  </x-tabbar>
  </div>
  
</template>

<script>
import Utils from "./common/js/utils";
import asCore from "./common/js/core";
import config from "./configs/tbconfig";
var modal = weex.requireModule("modal");
const navigator = weex.requireModule("navigator");
const loginBroad = new BroadcastChannel("login");
const themeBroad = new BroadcastChannel("theme");
const storage = weex.requireModule('storage'); 
export default {
  components: {
    "x-tabbar": require("./common/component/tabbar.vue"),
    "message": require("./yunagileoa/index/message.vue"),
    "appcenter": require("./yunagileoa/index/appcenter.vue"),
    "maillist": require("./yunagileoa/index/maillist.vue"),
    "my": require("./yunagileoa/index/my.vue")
  },
  computed: {
    showindex () {
      return indexShow;
    }
  },
  beforeCreate() {
    var s = this;
    var globalEvent = weex.requireModule("globalEvent");
    globalEvent.addEventListener("onPageInit", function(e) {
      const nav = weex.requireModule("navbar");
      nav.setStatusBarStyle("white");
      navigator.addBackGestureSelfControl();
      /* 包括已经存储的工作台展示方式 */
      s.loadTheme();
    });
    /* 判断是否登录过和是否超时(超时将自动从登) */
    asCore.getBsessionid(sessionid =>{
      if (!sessionid) {
        navigator.push(asCore.localpath + "login.js");
      } else {
        s.indexShow = true;
        /* 所有操作请在appinital 之后操作 */
        asCore.appInitial(() =>{
          /* 加载context */
          s.$refs.myinfo.isLogin();
        });
      }
    });
    /* 双击退出程序 */
    const page  = weex.requireModule("page");
    page.doubleBack();
    /* 登录之间进行通信的隧道 */
    loginBroad.onmessage = function(event) {
      var data = event.data;
      if(data.success == 1){
        page.reload();
      }
    };
    themeBroad.onmessage = function(event) {
        s.loadTheme();
    };
  },
  created() {
    Utils.initIconFont();
    const tabPageHeight = Utils.env.getPageHeight();
    const { tabStyles } = this;
    this.contentStyle = { height: tabPageHeight + 10 + "px" };
  },
  data() {
    return {
      tabTitles: config.tabIconFontTitles,
      tabStyles: config.tabIconFontStyles,
      indexShow: true,
      centerTheme:"lattice"
    };
  },
  methods: {
    wxcTabBarCurrentTabSelected(e) {
      const index = e.page;
      if(index==1){
         
      }
    },
    loadTheme (){
      var s = this;
      Utils.getThemeColor((color,theme) => {
        if(theme=="theme1"){
          s.tabStyles.activeIconFontColor = "#F15A49";
          s.tabStyles.activeTitleColor = "#F15A49";
          s.$refs.messageCom.setColor(color);
          
        }else{
          s.tabStyles.activeIconFontColor = color;
          s.tabStyles.activeTitleColor = color;
          s.$refs.messageCom.setColor(color);
        }
        s.$refs.messageCom.setTheme(theme);
        /* 去获取app应用的数据 */
        s.$refs.appcenterCom.getApp();

        s.$refs.maillistCom.setTheme(theme);
        s.$refs.appcenterCom.setTheme(theme);
        s.$refs.myinfo.setTheme(theme);
      });
      storage.getItem('appcenter', event => {
          if(event.result=="success"){
            s.centerTheme = event.data;
          }
      });

    }
  }
};
</script>
<style scoped>
.wrapper {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.item-container {
  flex: 1;
  width: 750px;
  background-color: #f5f5f5;
}
.test {
  height: 100px;
  width: 750px;
  background-color: #f5f5f5;
  align-items: center;
  justify-content: center;
}
</style>
