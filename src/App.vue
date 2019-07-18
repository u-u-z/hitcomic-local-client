<template>
  <div id="app">
    <!--
    <img src="./assets/logo.png">
    <div>
      <p>
        If Element is successfully added to this project, you'll see an
        <code v-text="'<el-button>'"></code>
        below
      </p>
      <el-button>el-button</el-button>
    </div>
    <HelloWorld msg="Welcome to Your Vue.js App"/>-->
    <el-row>
      <el-menu
        class="el-menu-demo"
        mode="horizontal"
        background-color="#2E8B57"
        text-color="#fff"
        style="padding:5px"
      >
        <h1 style="color:#FFF;">HITComic - 证件核销 - メイリン - 测试版！</h1>
      </el-menu>
    </el-row>
    <el-row justify="center" align="middle">
      <el-col :span="8">
        <el-alert title="入场实时图像捕捉区" type="info" center show-icon></el-alert>
        <video ref="video" id="video" width="320" height="240" autoplay></video>
      </el-col>
      <el-col :span="8">
        <el-alert title="入场照片捕获结果" type="info" center show-icon></el-alert>
        <canvas ref="canvas" id="canvas" width="320" height="240"></canvas>
      </el-col>

      <el-col :span="8">
        <el-alert title="入场照片记录" type="info" center show-icon></el-alert>
        <el-image :src="verifiedPictureUrl" width="320px" height="240px">
          <div slot="error" class="image-slot">
            <i class="el-icon-picture-outline">此处加载图片</i>
          </div>
        </el-image>
      </el-col>
    </el-row>
    <el-divider></el-divider>
    <el-row justify="center" align="middle" center show-icon>
      <el-col :span="24">
        <el-input id="ikey" placeholder="请输入内容" v-model="ticketKey" @keyup.enter.native="getCertInfo()">
          <template slot="prepend">请选中此处使用扫码枪扫码</template>
          <el-button slot="append" icon="el-icon-delete" v-on:click="ticketKey = ''"></el-button>
        </el-input>
      </el-col>
    </el-row>
    <el-divider content-position="center">
      <span style="color:#ccc;">工作人员操作区</span>
    </el-divider>
    <el-row justify="center" align="middle" center show-icon>
      <el-col :span="12">
        <el-button-group>
          <el-button type="primary" v-on:click="getCertInfo()">
            获取
            <i class="el-icon-sort-down el-icon--right"></i>
          </el-button>
          <el-button type="primary" v-on:click="capture()">
            拍照
            <i class="el-icon-camera el-icon--right"></i>
          </el-button>
          <el-button type="primary" v-on:click="verifyPost()">
            核销
            <i class="el-icon-finished el-icon--right"></i>
          </el-button>
          <el-button type="primary" v-on:click="clearCapture()">
            清空
            <i class="el-icon-delete el-icon--right"></i>
          </el-button>
        </el-button-group>
        <p></p>
      </el-col>
      <el-col :span="12">
        <p v-if="ticketKeytmp">当前操作的证件：{{ticketKeytmp}}</p>
      </el-col>
    </el-row>
    <el-divider content-position="center"> <span style="color:#ccc;">作者：蕾咪io 13093886624</span></el-divider>
  </div>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "app",
  /*components: {
    HelloWorld
  }*/
  data() {
    return {
      currentDate: new Date(),
      video: {},
      canvas: {},
      captures: [],
      videoElementHeight: 233,
      videoElementWidth: 233,
      certPircture: "",
      ticketKey: "",
      ticketKeytmp: "",
      serverUrl: "http://127.0.0.1:9090",
      verifiedPictureUrl: "",
      certPirctureBlob: ""
    };
  },
  methods: {
    capture() {
      this.canvas = this.$refs.canvas;
      //var context =
      this.canvas.getContext("2d").drawImage(this.video, 0, 0, 320, 240);
      //this.captures.push(this.canvas.toDataURL("image/jpeg"))
      this.certPircture = this.canvas.toDataURL("image/jpeg");
      this.focusOnInput()
    },

    clearCapture() {
      this.canvas = this.$refs.canvas;
      this.ctx = this.canvas.getContext("2d");
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.verifiedPictureUrl = "";
      this.ticketKeytmp = "";
      this.focusOnInput()
    },

    getCertInfo() {
      window.console.log("成功触发");
      this.ticketKeytmp = this.deepClone(this.ticketKey);
      fetch(`${this.serverUrl}/staff/${this.ticketKey}`)
        .then(response => response.json())
        .then(json => {
          window.console.log(json);
          if (this.notification(json["result"])) {
            window.console.log(json);
            if (json["quantity"] > 0) {
              window.console.log(
                json["records"][json["records"].length - 1]["Path"]
              );
              this.verifiedPictureUrl = `${this.serverUrl}/cert/${
                json["records"][json["records"].length - 1]["Path"]
              }.jpg`;
            } else {
              this.verifiedPictureUrl = ``;
            }
          } else {
            this.ticketKeytmp = "";
          }
        })
        .catch(error => {
          window.console.log("失敗" + error);
        });
      this.ticketKey = "";
      this.focusOnInput()
    },

    verifyPost() {
      this.canvas = this.$refs.canvas;
      this.canvas.toBlob(blob => {
        let formdata = new FormData();
        formdata.append("picture", blob);
        window.console.log(blob);
        if (blob.size < 1000) {
          this.$notify.error({
            title: "请采集照片",
            message: "您好像忘记了采集照片！"
          });
          this.focusOnInput()
          return;
        }
        fetch(`${this.serverUrl}/staff/${this.ticketKeytmp}`, {
          method: "POST",
          body: formdata
        })
          .then(response => response.json())
          .then(json => {
            window.console.log(json);
            if (json["result"] == "success") {
              this.$notify({
                title: "核销完成！✅",
                message: "有请下一位",
                type: "success",
                duration: 2000
              });
              
            } else if (this.notification(json["result"])) {
              window.console.log("没有核销成功!")
            } else {
              this.ticketKey = "";
            }
            this.clearCapture();
            this.focusOnInput()
          })
          .catch(error => {
            window.console.log("失敗" + error);
          });
      }, "image/jpeg");

      //this.clearCapture();
    },
    /**
     * 通知类型消息
     */

    notification(message) {
      switch (message) {
        case "success":
          this.$notify({
            title: "有效证件",
            message: "若有照片则显示在右侧",
            type: "success",
            duration: 2000
          });
          this.focusOnInput()
          return true;

        case "invalid":
          this.$notify.error({
            title: "次数用尽 🙅",
            message: "次数为 0 的证件无法使用"
          });
          this.focusOnInput()
          return false;

        case "fake":
          this.$notify.error({
            title: "虚假证件 🙅‍♂️",
            message: "证件不在数据库中"
          });
          this.focusOnInput()
          return false;

        case "fuckyou":
          this.$notify.error({
            title: "这TM是票！😠",
            message: "总之你可能进错口了"
          });
          this.focusOnInput()
          return false;

        default:
          this.$notify.info({
            title: "未知消息",
            message: "请联系管理员！错误信息" + message
          });
          this.focusOnInput()
          return false;
      }
    },
    deepClone(source) {
      return JSON.parse(JSON.stringify(source));
    },
    focusOnInput(){
      let importantInput = window.document.getElementById('ikey')
      importantInput.focus()
    }
  },
  mounted() {
    this.video = this.$refs.video;
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
        try {
          this.video.src = window.URL.createObjectURL(stream);
          this.video.play();
        } catch (error) {
          window.console.log("use new func.");
          this.video.srcObject = stream;
          this.video.play();
        }
      });
      this.videoElementHeight = this.$refs.video.offsetHeight;
      this.videoElementWidth = this.$refs.video.offsetWidth;
      this.focusOnInput()
    }
  }
};
</script>

<style>
#app {
  margin: 0;
  padding: 0;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  height: 1200;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}
.hit-button {
  width: 150px;
  height: 60px;
}
</style>
