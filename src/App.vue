<template>
  <div class="wrapper">
    <div></div>
    <h1 id="mytime">{{str}}</h1>
    <!-- <input id="mytime" type="text" name="" value="显示时间"> -->
    <div>
      <button id="stop" name="button" @click="stop">暂停</button>
      <button id="start" name="button" @click="start">开始</button>
      <button id="reset" name="button" @click="reset">重置</button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      h: 0, //定义时，分，秒，毫秒并初始化为0；
      m: 0,
      s: 0,
      ms: 0,
      time: 0,
      str: "00:00:00",
      mytime: "",
    };
  },
  methods: {
    timer() {
      //定义计时函数
      this.ms = this.ms + 50; //毫秒
      if (this.ms >= 1000) {
        this.ms = 0;
        this.s = this.s + 1; //秒
      }
      if (this.s >= 60) {
        this.s = 0;
        this.m = this.m + 1; //分钟
      }
      if (this.m >= 60) {
        this.m = 0;
        this.h = this.h + 1; //小时
      }
      this.str =
        this.toDub(this.h) +
        ":" +
        this.toDub(this.m) +
        ":" +
        this.toDub(this.s) +
        "" /*+this.toDubms(this.ms)+"毫秒"*/;
      // document.getElementById('mytime').innerHTML=h+"时"+m+"分"+s+"秒"+ms+"毫秒";
    },

    reset() {
      //重置
      clearInterval(this.time);
      this.h = 0;
      this.m = 0;
      this.s = 0;
      this.ms = 0;
      this.str = "00:00:00";
    },

    start() {
      //开始
      this.time = setInterval(this.timer, 50);
    },

    stop() {
      //暂停
      clearInterval(this.time);
    },

    toDub(n) {
      //补0操作
      if (n < 10) {
        return "0" + n;
      } else {
        return "" + n;
      }
    },

    toDubms(n) {
      //给毫秒补0操作
      if (n < 10) {
        return "00" + n;
      } else {
        return "0" + n;
      }
    },
  },
  //实例创建完成后调用，此阶段完成了数据的观测等，但尚未挂载，$el 还不可用。需要初始化处理一些数据时会比较有用
  created: function () {},
};
</script>
<style>
#mytime {
  background: red;
  color: yellow;
  display: block;
  font-size: 280px;
  height: 300px;
}
.wrapper {
  text-align: center;
  width: 100%;
  margin-top: 100px;
  /* margin: 250px auto; */
}
body {
  background-color: red;
}
button {
  padding: 0 20px;
  background-color: #2980ff;
  border: none;
  height: 34px;
  border-color: #2980ff;
  color: #fff;
  cursor: pointer;
  line-height: 34px;
  outline: none;
  -webkit-border-radius: 15px;
  -moz-border-radius: 15px;
  /* Old Firefox */
  border-radius: 15px;
  -webkit-transition: all 0.15s ease;
  -moz-transition: all 0.15s ease;
  -o-transition: all 0.15s ease;
  -ms-transition: all 0.15s ease;
  transition: all 0.15s ease;
  margin-right:20px;
}
button:hover {
  background-color: #0962e9;
}
</style>