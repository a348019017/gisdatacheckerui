<template>
  <div class="mainview">
    <el-tabs style="width:800px;height:600px" v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="主界面" name="first">

        <div class="upload-container">
          <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
            <el-form-item label="模板文件">
              <el-input v-model="formLabelAlign.temppath"></el-input>
              <el-button @click="openfiledialog(true)">导入模板("shp")</el-button>
              <el-button @click="openfolderdialog(true)">导入模板("gdb")</el-button>
            </el-form-item>
            <el-form-item label="对比文件">
              <el-input v-model="formLabelAlign.comparepath"></el-input>
              <el-button @click="openfiledialog(false)">导入比对文件("shp")</el-button>
              <el-button @click="openfolderdialog(false)">导入比对文件("gdb")</el-button>
            </el-form-item>

            <el-form-item label="">
              <el-button type="primary" size="medium " :disabled="isloading" @click="startCheck">开始对比</el-button>
            </el-form-item>

          </el-form>

          <!-- <el-upload class="upload-demo" drag action="" :before-upload="beforeAvatarUpload">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将模板文件拖到此处，或<em>点击上传</em></div>
            <div class="el-upload__tip" slot="tip">只能选取mdb/gdb/shp文件，且不超过1GB</div>
          </el-upload>

          <el-upload class="upload-demo" drag action="" :before-upload="beforeAvatarUpload">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将对比文件拖到此处，或<em>点击上传</em></div>
            <div class="el-upload__tip" slot="tip">只能选取mdb/gdb/shp文件，且不超过1GB</div>
          </el-upload>  -->
        </div>
      </el-tab-pane>
      <el-tab-pane label="配置管理" name="second">配置管理</el-tab-pane>
      <el-tab-pane label="检查结果" name="third">
        <el-table :data="dt" style="width: 100%">
          <el-table-column prop="type" label="错误类型" width="180">
          </el-table-column>
          <el-table-column prop="value" label="错误信息" width="180">
          </el-table-column>
          <el-table-column prop="alayers" label="模板图层">
          </el-table-column>
          <el-table-column prop="blayers" label="对比图层">
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>

//gdal并不成熟无法使用，仍旧采用
// import workerUrl from 'gdal3.js/dist/package/gdal3.js?url'
// import dataUrl from 'gdal3.js/dist/package/gdal3WebAssembly.data?url'
// import wasmUrl from 'gdal3.js/dist/package/gdal3WebAssembly.wasm?url'
// import initGdalJs from 'gdal3.js';
import * as WebSocket from "rpc-websockets"

export default {
  data() {
    return {
      activeName: "first",

      isloading: false,

      dt: [],
      istemp: true,
      labelPosition: 'right',
      formLabelAlign: {
        temppath: "D:\\WeChat Files\\WeChat Files\\xiezhipeng\\FileStorage\\File\\2023-10\\数据对比工具测试数据（不外传）\\矢量对比\\崇仁县自然资源登记单元矢量b\\崇仁县登记单元.shp",
        comparepath: `D:\\WeChat Files\\WeChat Files\\xiezhipeng\\FileStorage\\File\\2023-10\\数据对比工具测试数据（不外传）\\矢量对比\\崇仁县自然资源登记单元矢量a\\崇仁县登记单元.shp`,
      },
      checkrules: {
        valuecheck: {
          objectid: "OBJECTID",
        },
        filedscheck:{
        }
      }
    }
  },
  mounted() {
    this.ws = new WebSocket.Client('ws://localhost:8088')
    let that = this;
    this.ws.on('open', function () {
      that.ws.subscribe('openfiledialogresult');
      that.ws.subscribe('openfolderdialogresult')
      that.ws.on('openfiledialogresult', function (rst) {

        if (that.istemp) {
          that.formLabelAlign.temppath = rst;
        } else {
          that.formLabelAlign.comparepath = rst;
        }
      })
      that.ws.on('openfolderdialogresult', function (rst) {

        if (that.istemp) {
          that.formLabelAlign.temppath = rst;
        } else {
          that.formLabelAlign.comparepath = rst;
        }
      })
    });
  },

  methods: {
    async beforeAvatarUpload(file) {
      let that = this;
      return true;
    },
    startCheck() {
      if (this.formLabelAlign.temppath && this.formLabelAlign.comparepath) {
        let that = this;
        //this.isloading = true;
        //this.formLabelAlign.temppath, this.formLabelAlign.comparepath, this.checkrules
        this.ws.call('startcheck', {url1:this.formLabelAlign.temppath,url2:this.formLabelAlign.comparepath,checkrules:this.checkrules}).then(dt => {
          //this.isloading=false;
          that.$message.info("分析完成");
          that.activeName = "third";
          that.dt = dt;
        })

      } else {
        this.$message.info("请选择待分析的数据！");
        return;
      }
    },
    openfiledialog(istemp) {
      this.istemp = istemp;
      let that = this;
      that.ws.call('openfiledialog');
    },
    openfolderdialog(istemp) {
      this.istemp = istemp;
      let that = this;
      that.ws.call('openfolderdialog');
    },
  }




}
</script>



<style lang="less" scoped>
@media (min-width: 1024px) {
  .mainview {
    min-height: 100vh;
    height: 800px;
    display: flex;
    align-items: center;


    .upload-container {
      display: flex;

    }


  }
}
</style>
