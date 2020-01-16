<template>
  <el-dialog
          :visible="visible"
          @close="$emit('update:visible', false)"
          width="1282px"
          class="c-Result"
          :append-to-body="true"
  >
    <div class="dialog-title" slot="title">
    </div>
    <div
            v-for="(item, index) in resultList"
            :key="index"
            class="listrow"
            @click="
        event => {
          deleteRes(event, item);
        }
      "
    >
<!--      <div :visible="item.value && item.value.length === 0">-->
      <div class="name">
        {{ item.name }}
      </div>
      <div class="value">
        <span v-if="item.value && item.value.length === 0" class="card">
          ?
        </span>
        <span
                class="card"
                v-for="(data, j) in item.value"
                :key="j"
                :data-res="data">
          {{ list.find(d => d.key === data) ? list.find(d => d.key === data).name : 'Invalid' }}
        </span>
      </div>
<!--      </div>-->
    </div>
  </el-dialog>
</template>
<script>
  import { conversionCategoryName, getDomData } from '@/helper/index';
  export default {
    name: 'c-Result',
    props: {
      visible: Boolean
    },
    computed: {
      result: {
        get() {
          return this.$store.state.result;
        },
        set(val) {
          this.$store.commit('setResult', val);
        }
      },
      resultList() {
        const list = [];
        for (const key in this.result) {
          if (this.result.hasOwnProperty(key)) {
            const element = this.result[key];
            let name = conversionCategoryName(key);
            list.push({
              label: key,
              name,
              value: element
            });
          }
        }
        return list;
      },
      list() {
        return this.$store.state.list;
      }
    },
    methods: {
      deleteRes(event, row) {
        const Index = getDomData(event.target, 'res');
        if (!Index) {
          return;
        }
        this.$confirm('此操作将移除该中奖号码，确认删除?', '警告', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            if (Index) {
              const result = this.result;
              result[row.label] = this.result[row.label].filter(
                item => item !== Number(Index)
              );
              this.result = result;
              this.$message({
                type: 'success',
                message: '删除成功!'
              });
            }
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消'
            });
          });
      }
    }
  };
</script>
<style lang="scss">
  .el-dialog__header {
    padding: 0 !important;
  }
  .c-Result {
    .el-dialog {
      text-align: center;
      min-height: 650px;
      background-color: transparent;
      background-image: url('../assets/bg_results.png');
      background-size: 100% 100%;
      background-position: center center;
      background-repeat: no-repeat;
      /*top: 510px;*/
    }
    .listrow {
      display: block;
      /*line-height: 30px;*/
      margin-bottom: 30px;
      .name {
        //width: 120px;
        //font-weight: bold;
        font-size: 30pt;
        color: rgb(255, 96, 91);
      }
      .value {
        flex: 1;
        padding-left: 200px;
        padding-right: 200px;
      }
      .card {
        color: rgb(238, 229, 94);
        display: inline-block;
        //width: 40px;
        /*height: 40px;*/
        //line-height: 40px;
        text-align: center;
        font-size: 18px;
        /*font-weight: bold;*/
        /*border-radius: 4px;*/
        /*border: 1px solid #ccc;*/
        /*background-color: #f2f2f2;*/
        margin-left: 5px;
        margin-bottom: 5px;
        padding-left: 10px;
        padding-right: 10px;
        position: relative;
        cursor: pointer;
        &:hover {
          &::before {
            content: 'Del';
            width: 100%;
            height: 100%;
            background-color: #ccc;
            position: absolute;
            left: 0;
            top: 0;
            color: red;
            font-size: 12px;
          }
        }
      }
    }
  }
</style>
