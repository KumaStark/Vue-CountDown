<template>
  <div id="counterRoot" @click="checkClickLocation('background')">
    <div class="wrapper">
      <div id="settingButton">
        <el-button
          type="danger"
          size="mini"
          icon="el-icon-setting"
          @click="checkClickLocation('settingBtn')"
          :disabled="is_timer_started"
        ></el-button>
      </div>
      <div id="settingPanel" @click="checkClickLocation('settingPanel')">
        <transition name="el-fade-in-linear">
          <div v-show="showSettings" class="transition-box">
            <el-row :gutter="10">
              <el-col :span="6">
                <span>小时</span>
              </el-col>
              <el-col :span="6">
                <span>分钟</span>
              </el-col>
              <el-col :span="6">
                <span>秒数</span>
              </el-col>
              <el-col :span="6">
                <span>开关</span>
              </el-col>
            </el-row>
            <el-row :gutter="10">
              <el-col :span="6">
                <el-input-number
                  v-model="default_h"
                  @change="setDefault"
                  controls-position="right"
                  :min="0"
                  :max="99"
                ></el-input-number>
              </el-col>
              <el-col :span="6">
                <el-input-number
                  v-model="default_m"
                  @change="setDefault"
                  controls-position="right"
                  :min="0"
                  :max="59"
                ></el-input-number>
              </el-col>
              <el-col :span="6">
                <el-input-number
                  v-model="default_s"
                  @change="setDefault"
                  controls-position="right"
                  :min="0"
                  :max="59"
                ></el-input-number>
              </el-col>
              <el-col :span="6">
                <span>显示毫秒</span>
                <el-switch v-model="showMS" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
              </el-col>
            </el-row>
            <el-row :gutter="10" justify="space-between">
              <el-col :span="9">
                <el-button
                  class="preset-button"
                  type="danger"
                  icon="el-icon-data-board"
                  @click="preset1"
                >述职&论文</el-button>
              </el-col>
              <el-col :span="9">
                <el-button
                  class="preset-button"
                  type="danger"
                  icon="el-icon-s-comment"
                  @click="preset2"
                >抽题&答辩</el-button>
              </el-col>
              <el-col :span="6">
                <span>自动开始</span>
                <el-switch v-model="autoSt" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
              </el-col>
            </el-row>
          </div>
        </transition>
      </div>
      <h1 id="counterString">{{ time_str }}</h1>
      <div id="controlPanel">
        <el-button-group>
          <el-button
            type="danger"
            round
            @click="pause"
            icon="el-icon-video-pause"
            :disabled="sw_pause"
          >暂停</el-button>
          <el-button
            type="danger"
            round
            @click="start"
            icon="el-icon-video-play"
            :disabled="sw_start"
          >开始</el-button>
          <el-button
            type="danger"
            round
            @click="reset"
            icon="el-icon-refresh"
            :disabled="sw_reset"
          >重置</el-button>
        </el-button-group>
      </div>
    </div>
  </div>
</template>
<script>
document.title = "计时器";
export default {
  data() {
    return {
      showSettings: false,
      showMS: false,
      autoSt: true,
      default_h: 0,
      default_m: 10,
      default_s: 0,
      default_ms: 0,
      h: 0,
      m: 0,
      s: 0,
      ms: 0,
      timer_container: "TimerContainer",
      sw_start: false,
      sw_pause: true,
      sw_reset: false,
      is_timer_started: false,
      timeup_notifier: new Audio("./notify.wav"),
    };
  },
  computed: {
    time_str() {
      let check_time_remain = this.h + this.m + this.s + this.ms;
      if (check_time_remain > 0 && this.h > 0) {
        return (
          this.toDub(this.h) +
          ":" +
          this.toDub(this.m) +
          ":" +
          this.toDub(this.s)
        );
      } else if (check_time_remain > 0) {
        if (this.showMS) {
          return (
            this.toDub(this.m) +
            ":" +
            this.toDub(this.s) +
            "." +
            this.toDub(this.toDubms(this.ms))
          );
        } else {
          return this.toDub(this.m) + ":" + this.toDub(this.s);
        }
      } else {
        return "计时结束";
      }
    },
  },
  watch: {
    is_timer_started: {
      handler(newValue) {
        this.sw_pause = !newValue;
        if (this.time_str != "计时结束") {
          this.sw_start = newValue;
          // console.log("watch!this.time_str", this.time_str);
        }
        this.showSettings = false;
      },
      immediate: true,
    },
    time_str: {
      handler(newValue) {
        if (newValue == "计时结束") {
          if (this.is_timer_started) {
            this.pause();
            this.timeup_notifier.play();
          } else {
            this.sw_start = true;
          }
        } else if (!this.is_timer_started) {
          this.sw_start = false;
        }
      },
      immediate: true,
    },
  },
  methods: {
    preset1() {
      this.default_h = 0;
      this.default_m = 10;
      this.default_s = 0;
      this.default_ms = 0;
      this.setDefault();
      this.showSettings = false;
      if (this.autoSt) this.start();
    },
    preset2() {
      this.default_h = 0;
      this.default_m = 15;
      this.default_s = 0;
      this.default_ms = 0;
      this.setDefault();
      this.showSettings = false;
      if (this.autoSt) this.start();
    },
    checkClickLocation(clickFrom) {
      event.stopPropagation(); // 阻止点击穿透
      if (clickFrom == "settingBtn") {
        this.showSettings = !this.showSettings;
      } else if (clickFrom == "background") {
        this.showSettings = false;
      }
      // console.log("clickFrom", clickFrom);
    },
    setDefault() {
      this.h = this.default_h;
      this.m = this.default_m;
      this.s = this.default_s;
      this.ms = this.default_ms;
    },
    timer() {
      // 定义计时函数
      if (!this.showMS) {
        this.ms = this.ms + 1; //毫秒
        if (this.ms >= 100) {
          this.ms = 0;
          this.s = this.s - 1; //秒
        }
      } else {
        this.ms = this.ms - 1; //毫秒
        if (this.ms < 0) {
          this.ms = 99;
          this.s = this.s - 1; //秒
        }
      }
      if (this.s < 0) {
        this.s = 59;
        this.m = this.m - 1; //分钟
      }
      if (this.m < 0) {
        this.m = 59;
        this.h = this.h - 1; //小时
      }
    },
    reset() {
      //重置
      this.is_timer_started = false;
      clearInterval(this.timer_container);
      this.setDefault();
    },
    start() {
      //开始
      this.is_timer_started = true;
      this.timer_container = setInterval(this.timer, 10);
    },
    pause() {
      //暂停
      this.is_timer_started = false;
      clearInterval(this.timer_container);
    },
    // timeup() {
    //   this.is_timer_started = false;
    //   clearInterval(this.timer_container);
    //   this.sw_pause = true;
    //   this.sw_start = true;
    // },
    toDub(n) {
      //补0操作
      if (n < 10) {
        return "0" + n;
      } else {
        return "" + n;
      }
    },
    toDubms(n) {
      //给毫秒补位操作
      if (!this.showMS) {
        return 100 - n;
      } else {
        return n;
      }
    },
  },
  created: function () {
    this.setDefault();
  },
  mounted: function () {
    
  },
};
</script>
<style>
#counterRoot {
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;
}
#settingButton {
  position: fixed;
  top: 20px;
  left: 20px;
}

#settingPanel {
  position: fixed;
  display: flex;
  margin-top: 20px;
  height: 100px;
  top: 20px;
  left: 60px;
}

#counterString {
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
}
body {
  background-color: red;
}
.el-input-number {
  width: 100px !important;
}
.preset-button {
  width: 100%;
}
.transition-box {
  margin-bottom: 10px;
  width: 480px;
  height: 160px;
  border-radius: 4px;
  background-color: #409eff;
  text-align: center;
  color: #fff;
  padding: 20px;
  box-sizing: border-box;
  margin-right: 20px;
}

.el-row {
  margin-bottom: 10px;
}
:last-child {
  margin-bottom: 0;
}

.el-col {
  border-radius: 4px;
}
</style>
