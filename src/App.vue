<template>
  <div id="root">
    <header>
      <Publicity v-show="false" />
<!--      <el-button class="res" type="text" @click="showResult = true">-->
<!--        抽奖结果-->
<!--      </el-button>-->
<!--      <el-button class="con" type="text" @click="showConfig = true">-->
<!--        抽奖配置-->
<!--      </el-button>-->
    </header>
    <el-button id="globalConfigButton" icon="el-icon-setting" type="text" @click="showConfigCenter = true"></el-button>

    <el-link id="showResultButton" :underline="false" type="primary" @click="showResult = true">Lucky People</el-link>
    <div id="titleWrapper">
      <div class="logo"></div>
      <span>AILSH 2020 Annual Dinner</span>
    </div>

    <div id="main" :class="{ mask: showRes }"></div>
    <div id="tags">
      <ul v-for="item in datas" :key="item.key">
        <li>
          <a
            href="javascript:void(0);"
            :style="{
              color:
                !running && allresult.includes(item.key) ? '#ff2200' : '#fff'
            }"
          >
            {{ item.name ? item.name : item.key }}
            <img v-if="item.photo" :src="item.photo" :width="50" :height="50" />
          </a>
        </li>
      </ul>
    </div>
    <transition name="bounce">
      <div id="resbox" v-show="showRes">

        <div class="icon"></div>
        <p @click="showRes = false">{{ categoryName }}</p>


        <div class="container">
          <span
            v-for="item in resArr"
            :key="item"
            class="itemres"
            :style="resCardStyle"
            :data-id="item"
            @click="showRes = false"
            :class="{
              numberOver:
                !!photos.find(d => d.id === item) ||
                !!list.find(d => d.key === item)
            }"
          >
            <span class="cont" v-if="!photos.find(d => d.id === item)">
              <span
                v-if="!!list.find(d => d.key === item)"
                :style="{
                  fontSize: '40px'
                }"
              >
                {{ list.find(d => d.key === item).name }}
              </span>
              <span v-else>
                {{ item }}
              </span>
            </span>
            <img
              v-if="photos.find(d => d.id === item)"
              :src="photos.find(d => d.id === item).value"
              alt="photo"
              :width="160"
              :height="160"
            />
          </span>
        </div>
      </div>
    </transition>

<!--    <el-button-->
<!--      class="audio"-->
<!--      type="text"-->
<!--      @click="-->
<!--        () => {-->
<!--          playAudio(!audioPlaying);-->
<!--        }-->
<!--      "-->
<!--    >-->
<!--      <i-->
<!--        class="iconfont"-->
<!--        :class="[audioPlaying ? 'iconstop' : 'iconplay1']"-->
<!--      ></i>-->
<!--    </el-button>-->

    <LotteryConfig :visible.sync="showConfig" @resetconfig="reloadTagCanvas" />
    <Tool
      @toggle="toggle"
      @resetConfig="reloadTagCanvas"
      @showResult="showResultDialog"
      @getPhoto="getPhoto"
      :running="running"
      :closeRes="closeRes"
    />
    <Result :visible.sync="showResult"></Result>

<!--    <ConfigCenter :visible.sync="showConfigCenter"></ConfigCenter>-->
    <el-dialog
            :visible.sync="showConfigCenter"
            @close="$emit('update:visible', false)"
            width="600px"
            class="c-ConfigCenter"
            :append-to-body="true"
    >
      <div class="dialog-title" slot="title">
      <span :style="{ fontSize: '18px' }">
        Configuration
      </span>
      </div>
      <div>
        <el-button size="mini" @click="showImport = true">
          Import
        </el-button>
        <el-button size="mini" @click="showResult = true">
          Result
        </el-button>
<!--        <el-button size="mini" @click="showConfig = true">-->
<!--          Prize-->
<!--        </el-button>-->
        <el-button size="mini" @click="showRemoveoptions = true">
          Reset
        </el-button>
      </div>
    </el-dialog>
    <el-dialog
            :append-to-body="true"
            :visible.sync="showImport"
            class="import-dialog"
            width="400px"
    >
      <el-input
              type="textarea"
              :rows="10"
              placeholder="Please input names."
              v-model="listStr"
      ></el-input>
      <div class="footer">
        <el-button size="mini" type="primary" @click="transformList"
        >Save</el-button
        >
        <el-button size="mini" @click="showImport = false">Cancel</el-button>
      </div>
    </el-dialog>
    <el-dialog
            :visible.sync="showRemoveoptions"
            width="300px"
            class="c-removeoptions"
            :append-to-body="true"
    >
      <el-form ref="form" :model="removeInfo" label-width="80px" size="mini">
        <el-form-item label="Options">
          <el-radio-group v-model="removeInfo.type">
<!--            <el-radio border :label="0">Reset All</el-radio>-->
            <el-radio border :label="4">Reset Result</el-radio>
<!--            <el-radio border :label="1">Reset Prize Configuration</el-radio>-->
<!--            <el-radio border :label="2">Reset People List</el-radio>-->
<!--            <el-radio border :label="3">Reset Picture</el-radio>-->
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="resetConfig">Reset</el-button>
          <el-button @click="showRemoveoptions = false">Cancel</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
<!--    <span class="copy-right">-->
<!--      Copyright©zhangyongfeng5350@gmail.com-->
<!--    </span>-->

<!--    <audio-->
<!--      id="audiobg"-->
<!--      preload="auto"-->
<!--      controls-->
<!--      autoplay-->
<!--      loop-->
<!--      @play="playHandler"-->
<!--      @pause="pauseHandler"-->
<!--    >-->
<!--      <source :src="audioSrc" />-->
<!--      你的浏览器不支持audio标签-->
<!--    </audio>-->
  </div>
</template>
<script>
import LotteryConfig from '@/components/LotteryConfig';
import Publicity from '@/components/Publicity';
import Tool from '@/components/Tool';
import bgaudio from '@/assets/bg.mp3';
import beginaudio from '@/assets/begin.mp3';
import {
  getData,
  clearData,
  removeData,
  configField,
  resultField,
  newLotteryField,
  conversionCategoryName,
  listField,
  setData
} from '@/helper/index';
import { luckydrawHandler } from '@/helper/algorithm';
import Result from '@/components/Result';
// import ConfigCenter from "./components/ConfigCenter";
import { database, DB_STORE_NAME } from '@/helper/db';
export default {
  name: 'App',

  components: { LotteryConfig, Publicity, Tool, Result },

  computed: {
    resCardStyle() {
      const style = { fontSize: '30px' };
      // const { number } = this.config;
      style.fontSize = '40px';
      //
      // if (number < 100) {
      //   style.fontSize = '40px';
      // } else if (number < 1000) {
      //   style.fontSize = '40px';
      // } else if (number < 10000) {
      //   style.fontSize = '30px';
      // }
      return style;
    },
    config: {
      get() {
        return this.$store.state.config;
      }
    },
    result: {
      get() {
        return this.$store.state.result;
      },
      set(val) {
        this.$store.commit('setResult', val);
      }
    },
    list() {
      return this.$store.state.list;
    },
    allresult() {
      let allresult = [];
      for (const key in this.result) {
        if (this.result.hasOwnProperty(key)) {
          const element = this.result[key];
          allresult = allresult.concat(element);
        }
      }
      return allresult;
    },
    datas() {
      const datas = [];
      for (let index = 1; index <= this.config.number; index++) {
        const listData = this.list.find(d => d.key === index);
        const photo = this.photos.find(d => d.id === index);
        datas.push({
          key: index,
          name: listData ? listData.name : '',
          photo: photo ? photo.value : ''
        });
      }
      return datas;
    },
    categoryName() {
      return conversionCategoryName(this.category);
    },
    photos() {
      return this.$store.state.photos;
    }
  },
  created() {
    const data = getData(configField);
    if (data) {
      this.$store.commit('setConfig', Object.assign({}, data));
    }
    const result = getData(resultField);
    if (result) {
      this.$store.commit('setResult', result);
    }

    const newLottery = getData(newLotteryField);
    if (newLottery) {
      const config = this.config;
      newLottery.forEach(item => {
        this.$store.commit('setNewLottery', item);
        if (!config[item.key]) {
          this.$set(config, item.key, 0);
        }
      });
      this.$store.commit('setConfig', config);
    }

    const list = getData(listField);
    if (list) {
      this.$store.commit('setList', list);
    }
  },

  data() {
    return {
      running: false,
      showRes: false,
      showConfig: false,
      showRemoveoptions: false,
      showConfigCenter: false,
      showImport: false,
      showResult: false,
      resArr: [],
      category: '',
      audioPlaying: false,
      audioSrc: bgaudio,
      listStr: '',
      removeInfo: { type: 0 }
    };
  },
  watch: {
    photos: {
      deep: true,
      handler() {
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      }
    },
    showRemoveoptions(v) {
      if (!v) {
        this.removeInfo.type = 0;
      }
    }
  },
  mounted() {
    this.startTagCanvas();
    setTimeout(() => {
      this.getPhoto();
    }, 1000);

    // window.addEventListener('keyup',this.handleKeyup)
  },
  methods: {

    resetConfig() {
      const type = this.removeInfo.type;
      this.$confirm('Please double confirm your operation?', 'Warning', {
        confirmButtonText: 'Yes',
        cancelButtonText: 'Cancel',
        type: 'warning'
      })
        .then(() => {
          switch (type) {
            case 0:
              clearData();
              this.$store.commit('setClearStore');
              database.clear(DB_STORE_NAME);
              break;
            case 1:
              removeData(configField);
              this.$store.commit('setClearConfig');
              break;
            case 2:
              removeData(listField);
              this.$store.commit('setClearList');
              break;
            case 3:
              database.clear(DB_STORE_NAME);
              this.$store.commit('setClearPhotos');
              break;
            case 4:
              removeData(resultField);
              this.$store.commit('setClearResult');
              break;
            default:
              break;
          }

          this.closeRes && this.closeRes();

          this.showRemoveoptions = false;
          this.$message({
            type: 'success',
            message: '重置成功!'
          });

          this.$nextTick(() => {
            //this.$emit('resetConfig');
            this.reloadTagCanvas()
          });
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });
        });
    },
    transformList() {
      setData(configField, this.$store.state.config);

      const { listStr } = this;
      if (!listStr) {
        this.$message.error('没有数据');
      }
      const list = [];
      const rows = listStr.split('\n');
      let rowNumber = 0;
      if (rows && rows.length > 0) {
        rows.forEach(item => {
          if(item === '') {
            return;
          }
          rowNumber++;
          list.push({
            key: rowNumber,
            name: item
          });

          // const rowList = item.split(/\t|\s/);
          // if (rowList.length >= 2) {
          //   const key = Number(rowList[0].trim());
          //   const name = rowList[1].trim();
          //   key &&
          //   list.push({
          //     key,
          //     name
          //   });
          // }
        });
      }
      this.$store.commit('setList', list);

      let config = this.$store.state.config;
      config.number = rowNumber;
      this.$store.commit('setConfig', config);

      let configInStorage = getData(configField);
      configInStorage.number = rowNumber;
      setData(configField, configInStorage);

      this.$message({
        message: '保存成功',
        type: 'success'
      });

      this.showImport = false;
      this.$nextTick(() => {
        // this.$emit('resetConfig');
        this.reloadTagCanvas();
      });
    },
    playHandler() {
      this.audioPlaying = true;
    },
    pauseHandler() {
      this.audioPlaying = false;
    },
    playAudio(type) {
      if (type) {
        // this.$el.querySelector('#audiobg').play();
      } else {
        // this.$el.querySelector('#audiobg').pause();
      }
    },
    loadAudio() {
      // this.$el.querySelector('#audiobg').load();
      this.$nextTick(() => {
        // this.$el.querySelector('#audiobg').play();
      });
    },
    getPhoto() {
      database.getAll(DB_STORE_NAME).then(res => {
        if (res && res.length > 0) {
          this.$store.commit('setPhotos', res);
        }
      });
    },
    speed() {
      return [0.1 * Math.random() + 0.01, -(0.1 * Math.random() + 0.01)];
    },
    createCanvas() {
      const canvas = document.createElement('canvas');
      canvas.width = document.body.offsetWidth;
      canvas.height = document.body.offsetHeight;
      canvas.id = 'rootcanvas';
      this.$el.querySelector('#main').appendChild(canvas);
    },
    startTagCanvas() {
      this.createCanvas();
      const { speed } = this;
      window.TagCanvas.Start('rootcanvas', 'tags', {
        textColour: null,
        initial: speed(),
        dragControl: 1,
        textHeight: 20,
        noSelect: true,
        lock: 'xy',
        wheelZoom: false
      });
    },
    reloadTagCanvas() {
      window.TagCanvas.Reload('rootcanvas');
    },
    showResultDialog() {
      this.showResult = true;
    },
    closeRes() {
      this.showRes = false;
    },
    toggle(form) {
      const { speed, config } = this;
      if (this.running) {
        this.audioSrc = bgaudio;
        this.loadAudio();

        window.TagCanvas.SetSpeed('rootcanvas', speed());
        this.showRes = true;
        this.running = !this.running;
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      } else {
        this.showRes = false;
        if (!form) {
          return;
        }

        this.audioSrc = beginaudio;
        this.loadAudio();

        const { number } = config;
        const { category, mode, qty, remain, allin } = form;
        let num = 1;
        if (mode === 1 || mode === 5) {
          num = mode;
        } else if (mode === 0) {
          num = remain;
        } else if (mode === 99) {
          num = qty;
        }
        const resArr = luckydrawHandler(
          number,
          allin ? [] : this.allresult,
          num
        );
        this.resArr = resArr;

        this.category = category;
        if (!this.result[category]) {
          this.$set(this.result, category, []);
        }
        const oldRes = this.result[category] || [];
        const data = Object.assign({}, this.result, {
          [category]: oldRes.concat(resArr)
        });
        this.result = data;
        window.TagCanvas.SetSpeed('rootcanvas', [0.6, 1.2]);
        this.running = !this.running;
      }
    }
  }
};
</script>
<style lang="scss">
#root {
  height: 100%;
  position: relative;
  background-image: url('./assets/bg.jpg');
  /*background-size: 50% 50%;*/
  background-size: 100% 100%;
  background-position: center center;
  background-repeat: no-repeat;
  background-color: #000000;
  /*background-color: #121936;*/
  .mask {
    -webkit-filter: blur(5px);
    filter: blur(5px);
  }
  header {
    /*height: 50px;*/
    line-height: 50px;
    position: relative;
    .el-button {
      position: absolute;
      top: 17px;
      padding: 0;
      z-index: 9999;
      &.con {
        right: 20px;
      }
      &.res {
        right: 100px;
      }
    }
  }
  .audio {
    position: absolute;
    top: 100px;
    right: 30px;
    width: 40px;
    height: 40px;
    line-height: 40px;
    border: 1px solid #fff;
    border-radius: 50%;
    padding: 0;
    text-align: center;
    .iconfont {
      position: relative;
      left: 1px;
    }
  }
  .copy-right {
    position: absolute;
    right: 0;
    bottom: 0;
    color: #ccc;
    font-size: 12px;
  }
  .bounce-enter-active {
    animation: bounce-in 1.5s;
  }
  .bounce-leave-active {
    animation: bounce-in 0s reverse;
  }
}
#main {
  height: 100%;
}

#main canvas {
  padding-top: 15px;
}

#resbox {
  position: absolute;
  top: 425px;
  left: 50%;
  width: 1040px;
  height: 636px;
  transform: translateX(-50%) translateY(-50%);
  background-image: url("./assets/bg_Lucky-draw.png");
  background-size: 100% 100%;
  background-position: center center;
  background-repeat: no-repeat;
  text-align: center;

  .icon {
    position: absolute;
    width: 103px;
    height: 81px;
    left: 320px;
    top: 25px;
    background-image: url("./assets/icon_prize.png");
    background-size: 100% 100%;
    background-position: center center;
    background-repeat: no-repeat;
  }
  p {
    display: block;
    position: absolute;
    left: 455px;
    top: 50px;
    color: rgb(255, 96, 91);
    font-size: 35px;
    //line-height: 120px;
  }


  .container {
    display: block;
    justify-content: center;
    flex-wrap: wrap;
    padding-left: 400px;
    padding-top: 150px;
  }
  .itemres {
    background-image: url("./assets/bg_name.png");
    background-size: 100% 100%;
    background-position: center center;
    background-repeat: no-repeat;
    color: rgb(238, 229, 94);
    //background: #fff;
    /*width: 160px;*/
    height: 68px;
    width: 223px;
    //border-radius: 4px;
    //border: 1px solid #ccc;
    //line-height: 160px;
    //font-weight: bold;
    margin-right: 20px;
    margin-bottom: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    .cont {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    &.numberOver::before {
      /*content: attr(data-id);*/
      width: 30px;
      height: 22px;
      //line-height: 22px;
      background-color: #fff;
      position: absolute;
      bottom: 0;
      left: 0;
      font-size: 14px;
      // border-radius: 50%;
      z-index: 1;
    }
  }
}
#globalConfigButton {
  position: absolute;
  right: 10px;
  bottom: 0px;
  z-index: 2000;
  font-size: 20px;
}
#showResultButton {
  position: absolute;
  bottom: 10px;
  left: 0;
  right: 0;
  margin-left: auto;
  margin-right: auto;
  z-index: 1000;
  font-size: 14px;
  width: 200px;
}
#titleWrapper {
  position: absolute;
  top: 30px;
  left: 0;
  right: 0;
  margin-left: auto;
  margin-right: auto;
  z-index: 1000;
  width: 690px;
  height: 100px;
  /*font-family: Verdana;*/
  font-size: 30pt;
  color: #ffffff;
  /*background-color: white;*/
}
#titleWrapper .logo {
  width: 128px;
  height: 61px;
  background-image: url('./assets/logo.png');
  background-size: 80% 80%;
  background-position: center center;
  background-repeat: no-repeat;
}
#titleWrapper span {
  position: absolute;
  left: 146px;
  top: 5px;
}
.v-modal {
  opacity: 0.8 !important;
}
</style>
