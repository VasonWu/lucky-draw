<template>
  <div id="tool">
<!--    <el-button v-if="!running && remainFirstPrize > 0 &&remainSecondPrize === 0" size="mini" @click="onStartFirstPrize" :disabled="remainFirstPrize == 0 || remainSecondPrize > 0">-->
<!--      First Prize [{{remainFirstPrize}}]-->
<!--    </el-button>-->
<!--    <el-button v-if="!running && remainSecondPrize > 0 && remainThirdPrize === 0" size="mini" @click="onStartSecondPrize" :disabled="remainSecondPrize == 0 || remainThirdPrize > 0">-->
<!--      Second Prize [{{remainSecondPrize}}]-->
<!--    </el-button>-->
<!--    <el-button v-if="!running && remainThirdPrize > 0 && remainFourthPrize === 0" size="mini" @click="onStartThirdPrize" :disabled="remainThirdPrize == 0 || remainFourthPrize > 0">-->
<!--      Third Prize [{{remainThirdPrize}}]-->
<!--    </el-button>-->
<!--    <el-button v-if="!running && remainFourthPrize > 0" size="mini" @click="onStartFourthPrize" :disabled="remainFourthPrize == 0">-->
<!--      Fourth Prize [{{remainFourthPrize}}]-->
<!--    </el-button>-->
<!--    <el-button v-if="!running && remainFirstPrize === 0" @click="showResult" type="primary" size="mini">Show Result</el-button>-->
<!--    <el-button v-if="running" @click="startHandler" type="primary" size="mini">Stop</el-button>-->
<!--    <el-button size="mini" @click="showRemoveoptions = true">-->
<!--      重置-->
<!--    </el-button>-->
<!--    <el-button size="mini" @click="showImport = true">-->
<!--      导入名单-->
<!--    </el-button>-->
<!--    <el-button size="mini" @click="showImportphoto = true">-->
<!--      导入照片-->
<!--    </el-button>-->
    <el-dialog
            :append-to-body="true"
            :visible.sync="showSetwat"
            class="setwat-dialog"
            width="400px"
    >
      <el-form ref="form" :model="form" label-width="80px" size="mini">
        <el-form-item label="抽取奖项">
          <el-select v-model="form.category" placeholder="请选取本次抽取的奖项">
            <el-option
                    :label="item.label"
                    :value="item.value"
                    v-for="(item, index) in categorys"
                    :key="index"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label=" " v-if="form.category">
          <span>
            共&nbsp;
            <span class="colorred">{{ config[form.category] }}</span>
            &nbsp;名
          </span>
          <span :style="{ marginLeft: '20px' }">
            剩余&nbsp;
            <span class="colorred">{{ remain }}</span>
            &nbsp;名
          </span>
        </el-form-item>

        <el-form-item label="抽取方式">
          <el-select v-model="form.mode" placeholder="请选取本次抽取方式">
            <el-option label="抽1人" :value="1"></el-option>
            <el-option label="抽5人" :value="5"></el-option>
            <el-option label="一次抽取完" :value="0"></el-option>
            <el-option label="自定义" :value="99"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="抽取人数" v-if="form.mode === 99">
          <el-input
                  v-model="form.qty"
                  type="number"
                  :clearable="true"
                  :min="1"
                  :max="remain ? remain : 100"
                  :step="1"
          ></el-input>
        </el-form-item>

        <el-form-item label="全员参与">
          <el-switch v-model="form.allin"></el-switch>
          <span :style="{ fontSize: '12px' }">
            (开启后将在全体成员[无论有无中奖]中抽奖)
          </span>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="onSubmit">立即抽奖</el-button>
          <el-button @click="showSetwat = false">取消</el-button>
        </el-form-item>
      </el-form>
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
              placeholder="请输入名单，每行一个名字"
              v-model="listStr"
      ></el-input>
      <div class="footer">
        <el-button size="mini" type="primary" @click="transformList"
        >确定</el-button
        >
        <el-button size="mini" @click="showImport = false">取消</el-button>
      </div>
    </el-dialog>
    <Importphoto
            :visible.sync="showImportphoto"
            @getPhoto="$emit('getPhoto')"
    ></Importphoto>

    <el-dialog
            :visible.sync="showRemoveoptions"
            width="300px"
            class="c-removeoptions"
            :append-to-body="true"
    >
      <el-form ref="form" :model="removeInfo" label-width="80px" size="mini">
        <el-form-item label="重置选项">
          <el-radio-group v-model="removeInfo.type">
            <el-radio border :label="0">重置全部数据</el-radio>
            <el-radio border :label="1">重置抽奖配置</el-radio>
            <el-radio border :label="2">重置名单</el-radio>
            <el-radio border :label="3">重置照片</el-radio>
            <el-radio border :label="4">重置抽奖结果</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="resetConfig">确定重置</el-button>
          <el-button @click="showRemoveoptions = false">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
  import {
    clearData,
    removeData,
    configField,
    listField,
    resultField,
    getData,
    setData,
    conversionCategoryName
  } from '@/helper/index';
  import Importphoto from './Importphoto';
  import { database, DB_STORE_NAME } from '@/helper/db';

  export default {
    props: {
      running: Boolean,
      closeRes: Function
    },
    computed: {
      config: {
        get() {
          return this.$store.state.config;
        }
      },
      remainFirstPrize() {
        return this.config['firstPrize'] - (this.result['firstPrize'] ? this.result['firstPrize'].length : 0);
      },
      remainSecondPrize() {
        return this.config['secondPrize'] - (this.result['secondPrize'] ? this.result['secondPrize'].length : 0);
      },
      remainThirdPrize() {
        return this.config['thirdPrize'] - (this.result['thirdPrize'] ? this.result['thirdPrize'].length : 0);
      },
      remainFourthPrize() {
        return this.config['fourthPrize'] - (this.result['fourthPrize'] ? this.result['fourthPrize'].length : 0);
      },
      remain() {
        return (
          this.config[this.form.category] -
          (this.result[this.form.category]
            ? this.result[this.form.category].length
            : 0)
        );
      },
      result() {
        return this.$store.state.result;
      },
      categorys() {
        const options = [];
        for (const key in this.config) {
          if (this.config.hasOwnProperty(key)) {
            const item = this.config[key];
            if (item > 0) {
              let name = conversionCategoryName(key);
              name &&
              options.push({
                label: name,
                value: key
              });
            }
          }
        }
        return options;
      }
    },
    components: { Importphoto },
    mounted() {
      window.addEventListener('keyup',this.handleKeyup)
    },
    data() {
      return {
        showSetwat: false,
        showImport: false,
        showImportphoto: false,
        showRemoveoptions: false,
        allowStartNextLevelPrize: true,
        removeInfo: { type: 0 },
        form: {
          category: '',
          mode: 1,
          qty: 1,
          allin: false
        },
        listStr: '',
        lastPrizeTime: null
      };
    },
    watch: {
      showRemoveoptions(v) {
        if (!v) {
          this.removeInfo.type = 0;
        }
      }
    },
    methods: {
      startNextPrize() {
        // if(this.running) {
        //   this.lastPrizeTime = null;
        //   this.startHandler();
        //   return;
        // }

        if(!this.running && this.remainFourthPrize > 0) {
          this.onStartFourthPrize();
          return;
        } else if(this.running) {
          this.startHandler();
          this.lastPrizeTime = Date.now();
          return;
        }

        if(!this.running && this.remainThirdPrize > 0) {
          if(this.remainThirdPrize == this.config['thirdPrize']) {
            this.$confirm('Current round lucky draw is finished. Do you want to start next round lucky draw?', 'Congratulations!', {
              confirmButtonText: 'Yes',
              cancelButtonText: 'Cancel',
              type: 'info'
            }).then(() => {
              this.onStartThirdPrize();
              return;
            })
          } else {
            this.onStartThirdPrize();
          }

          return;
        } else if(this.running) {
          this.startHandler();
          this.lastPrizeTime = Date.now();
          return;
        }

        if(!this.running && this.remainSecondPrize > 0) {
          if(this.remainSecondPrize == this.config['secondPrize']) {
            this.$confirm('Current round lucky draw is finished. Do you want to start next round lucky draw?', 'Congratulations!', {
              confirmButtonText: 'Yes',
              cancelButtonText: 'Cancel',
              type: 'info'
            }).then(() => {
              this.onStartSecondPrize();
              return;
            })
          } else {
            this.onStartSecondPrize();
          }

          return;
        } else if(this.running) {
          this.startHandler();
          this.lastPrizeTime = Date.now();
          return;
        }

        if(!this.running && this.remainFirstPrize > 0) {
          if(this.remainFirstPrize == this.config['firstPrize']) {
            this.$confirm('Current round lucky draw is finished. Do you want to start next round lucky draw?', 'Congratulations!', {
              confirmButtonText: 'Yes',
              cancelButtonText: 'Cancel',
              type: 'info'
            }).then(() => {
              this.onStartFirstPrize();
              return;
            })
          } else {
            this.onStartFirstPrize();
          }

          return;
        } else if(this.running) {
          this.startHandler();
          this.lastPrizeTime = Date.now();
          return;
        }

        this.showResult();
      },
      handleKeyup(event) {
        const e = event || window.event || arguments.callee.caller.arguments[0]
        if(!e) return
        if(e.keyCode === 32) {
          this.startNextPrize();
        }
        // const {key,keyCode} = e
        // console.log(e.keyCode)
        // console.log(key)
      },
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
              this.$emit('resetConfig');
            });
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消'
            });
          });
      },
      onStartFirstPrize() {
        let remain =  this.config['firstPrize'] - (this.result['firstPrize'] ? this.result['firstPrize'].length : 0);
        if(remain <= 0) {
          return this.$message.error('该奖项已抽完');
        }
        this.$emit(
          'toggle',
          Object.assign({}, {
            allin: false,
            category: 'firstPrize',
            mode: 1,
            qty: 0,
            remain: 0
          })
        );
      },
      onStartSecondPrize() {
        let remain =  this.config['secondPrize'] - (this.result['secondPrize'] ? this.result['secondPrize'].length : 0);
        if(remain <= 0) {
          return this.$message.error('该奖项已抽完');
        }

        this.$emit(
          'toggle',
          Object.assign({}, {
            allin: false,
            category: 'secondPrize',
            mode: 1,
            qty: 0,
            remain: 0
          })
        );
      },
      onStartThirdPrize() {
        let remain =  this.config['thirdPrize'] - (this.result['thirdPrize'] ? this.result['thirdPrize'].length : 0);
        if(remain <= 0) {
          return this.$message.error('该奖项已抽完');
        }

        let qty = 2;
        if((remain - qty) < 0) {
          qty = remain
        }

        this.$emit(
          'toggle',
          Object.assign({}, {
            allin: false,
            category: 'thirdPrize',
            mode: 99,
            qty: qty,
            remain: 0
          })
        );
      },
      onStartFourthPrize() {
        let remain =  this.config['fourthPrize'] - (this.result['fourthPrize'] ? this.result['fourthPrize'].length : 0);
        if(remain <= 0) {
          return this.$message.error('该奖项已抽完');
        }

        let qty = 5;
        if(remain >= 17) {
          qty = 5;
        } else {
          qty = 4;
        }

        if((remain - qty) < 0) {
          qty = remain
        }

        this.$emit(
          'toggle',
          Object.assign({}, {
            allin: false,
            category: 'fourthPrize',
            mode: 99,
            qty: qty,
            remain: 0
          })
        );
      },
      onSubmit() {
        if (!this.form.category) {
          return this.$message.error('请选择本次抽取的奖项');
        }
        if (this.remain <= 0) {
          return this.$message.error('该奖项剩余人数不足');
        }
        if (this.form.mode === 99) {
          if (this.form.qty <= 0) {
            return this.$message.error('必须输入本次抽取人数');
          }
          if (this.form.qty > this.remain) {
            return this.$message.error('本次抽奖人数已超过本奖项的剩余人数');
          }
        }
        if (this.form.mode === 1 || this.form.mode === 5) {
          if (this.form.mode > this.remain) {
            return this.$message.error('本次抽奖人数已超过本奖项的剩余人数');
          }
        }
        this.showSetwat = false;
        this.$emit(
          'toggle',
          Object.assign({}, this.form, { remain: this.remain })
        );
      },
      startHandler() {
        this.$emit('toggle');
        if (!this.running) {
          this.showSetwat = true;
        }
      },
      showResult() {
        this.$emit('showResult');
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
          this.$emit('resetConfig');
        });
      }
    }
  };
</script>
<style lang="scss">
  #tool {
    position: fixed;
    width: 60px;
    height: 500px;
    top: 50%;
    right: 50px;
    transform: translateY(-50%);
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    .el-button + .el-button {
      margin-top: 20px;
      margin-left: 0px;
    }
  }
  .setwat-dialog {
    .colorred {
      color: red;
      font-weight: bold;
    }
  }
  .import-dialog {
    .footer {
      height: 50px;
      line-height: 50px;
      text-align: center;
    }
  }
  .c-removeoptions {
    .el-dialog {
      height: 290px;
    }
    .el-radio.is-bordered + .el-radio.is-bordered {
      margin-left: 0px;
    }
    .el-radio.is-bordered {
      margin-bottom: 10px;
    }
  }
</style>
