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
        <h1 style="color:#FFF;">HIT动漫 - 证件核销系统 - ホン メイリン</h1>
      </el-menu>
    </el-row>
    <el-row justify="center" align="middle">
      <el-col :span="8">
        <el-alert title="入场实时图像捕捉区" type="info" center show-icon></el-alert>
        <video ref="video" id="video" width="100%" autoplay></video>
      </el-col>
      <el-col :span="8">
        <el-alert title="入场照片捕获结果" type="info" center show-icon></el-alert>
      </el-col>

      <el-col :span="8">
        <el-alert title="入场照片记录" type="info" center show-icon></el-alert>
      </el-col>
    </el-row>
    <el-row justify="center" align="middle" center show-icon>
      <el-col :xs="24" :sm="24" :md="24" :lg="24" :xl="24">3</el-col>
    </el-row>
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
      captures: []
    };
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
</style>
