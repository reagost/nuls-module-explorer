<template>
  <div class="code-info bg-white">
    <div class="certified" v-if="ifCertified">
      <div class="top-info">
        <h5 class="font16 fw fl"><i class="el-icon-success fCN"></i>{{$t('codeInfo.codeInfo0')}}</h5>
        <div class="fr">
          <span class="font12">{{$t('codeInfo.codeInfo1')}}：NVM V1.0.0 </span>
          <span class="font12">{{$t('codeInfo.codeInfo2')}}：{{certificationTime ==='null' ? certificationTimes: certificationTime}}</span>
        </div>
      </div>
      <div class="code-list">
        <div class="code-tree fl">
          <div class="code-tree-title">
            <h6 class="font14 fl">{{$t('codeInfo.codeInfo3')}}</h6>
            <i class="el-icon-download fr click"></i>
          </div>
          <div class="code-trees">
            <el-tree :data="codeTress" :props="defaultProps" @node-click="handleNodeClick"
                     :default-expand-all="true"></el-tree>
          </div>
        </div>
        <div class="code-source fl">
          <div class="tr code-bottom"><i class="iconfont icon-copy_icon click" @click="copy(codeInfo)"></i></div>
          <div class="code-source-info bg-gray">
            <pre>{{codeInfo}}</pre>
          </div>
        </div>
      </div>
    </div>

    <div v-if="!ifCertified" class="certified certifing" v-loading="uploadLoading">
      <div class="top-info">
        <h5 class="font16 fw fl"><i class="el-icon-warning fCN"></i>{{$t('codeInfo.codeInfo4')}}</h5>
      </div>
      <div class="upload tc" v-show="true">
        <input type="file" id="fileId" class="hidden file">
        <el-button type="success" @click="uploadFile">{{$t('codeInfo.codeInfo6')}} <i
                class="el-icon-upload el-icon&#45;&#45;right"></i>
        </el-button>
        <div class="upload-info tl font14">
          <p>{{$t('codeInfo.codeInfo7')}}:</p>
          <p>{{$t('codeInfo.codeInfo8')}}</p>
          <p>{{$t('codeInfo.codeInfo9')}}</p>
          <p>{{$t('codeInfo.codeInfo13')}}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import moment from 'moment'
  import {copys, getLocalTime} from '@/api/util.js'
  import {CODE_URL} from './../../config'

  export default {
    props: {status: Number, certificationTime: String},
    data() {
      return {
        //是否已认证
        ifCertified: false,
        //认证时间
        certificationTimes: '',
        //合约地址
        contractsAddress: this.$route.query.contractAddress,

        //上传源代码动画
        uploadLoading: false,

        //代码目录
        codeTress: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        },

        //代码内容
        codeInfo: '',
      };
    },
    watch:{
      status:{
        handler(newval){
          this.ifCertified = newval === 2;
          if (this.ifCertified) {
            this.getContractCodeTree(this.contractsAddress);
          }
        },
        immediate: true
      }
    },
    
    methods: {
      /**
       * 获取zip文件转换为文件流
       **/
      uploadFile() {
        let _this = this;
        let obj = document.getElementById("fileId");
        obj.click();
        obj.onchange = function () {
          if (this.value !== '') {
            let file = obj.files[0];
            //获取文件流
            let reader = new FileReader();
            let total = file.size;
            if (total > 1024 * 1024 * 0.5) {
              _this.$message({message: _this.$t('codeInfo.codeInfo10'), type: 'error', duration: 3000});
              return;
            }
            let jobSpecFileDataURL = '';
            reader.readAsDataURL(file);
            reader.onload = (() => {
              jobSpecFileDataURL = reader.result;
              _this.uploadLoading = true;
              _this.uploadFiles(_this.contractsAddress, jobSpecFileDataURL);
            });
          }
        }
      },

      /**
       * 调用认证方法
       **/
      async uploadFiles(contractsAddress, jobSpecFile) {
        const params = {
          "jsonrpc": "2.0",
          "method": 'validateContractCode',
          "params": [Number(sessionStorage.getItem('chainId')), contractsAddress, jobSpecFile],
          "id": Math.floor(Math.random() * 1000)
        };
        axios.post(CODE_URL, params)
          .then((response) => {
            //console.log(response.data);
            if (response.data.result) {
              this.ifCertified = true;
              this.getContractCodeTree(contractsAddress);
              this.contractStatus();
              let timestamp = (new Date()).getTime();
              this.certificationTimes = moment(getLocalTime(timestamp)).format('YYYY-MM-DD HH:mm:ss');
              this.uploadLoading = false;
            } else {
              this.$message({
                message: this.$t('codeInfo.codeInfo5') + response.data.error.message,
                type: 'error',
                duration: 1500
              });
              this.uploadLoading = false;
            }
          }).catch((error) => {
          console.log(error);
          this.$message({message: this.$t('codeInfo.codeInfo11'), type: 'error', duration: 2000});
          this.uploadLoading = false;
        })
      },

      /**
       * 认证完成传参给父组件
       **/
      contractStatus() {
        this.$emit('contractStatus', 2)
      },

      /**
       * 获取合约代码目录
       **/
      async getContractCodeTree(contractsAddress) {
        const params = {
          "jsonrpc": "2.0",
          "method": 'getContractCodeTree',
          "params": [Number(sessionStorage.getItem('chainId')), contractsAddress],
          "id": Math.floor(Math.random() * 1000)
        };
        axios.post(CODE_URL, params)
          .then((response) => {
            //console.log(response);
            if (response.data.result) {
              this.codeTress.push(response.data.result.root);
            }
          }).catch((error) => {
          console.log(error)
        })
      },

      /**
       * 获取合约代码
       **/
      async getContractCode(contractsAddress, path) {
        const params = {
          "jsonrpc": "2.0",
          "method": 'getContractCode',
          "params": [Number(sessionStorage.getItem('chainId')), contractsAddress, path],
          "id": Math.floor(Math.random() * 1000)
        };
        axios.post(CODE_URL, params)
          .then((response) => {
            //console.log(response);
            if (response.data.hasOwnProperty("result")) {
              this.codeInfo = response.data.result;
            } else {
              this.codeInfo = '';
            }
          }).catch((error) => {
          console.log(error)
        })
      },

      /**
       * 点击目录获取代码
       * @param data
       */
      handleNodeClick(data) {
        //console.log(data);
        if (!data.dir) {
          //console.log(data.path);
          this.getContractCode(this.contractsAddress, data.path);
        }
      },

      /**
       * 复制方法
       * @param sting
       **/
      copy(sting) {
        copys(sting);
      },

    }
  };
</script>

<style lang="less">
  @import "./../../assets/css/style";

  .code-info {
    min-height: 500px;
    margin: 10px auto 0;
    border: @BD1;
    .certified {
      .top-info {
        height: 50px;
        line-height: 50px;
        border-bottom: @BD1;
        margin: 0 20px;
        h5 {
          line-height: 50px;
          margin-left: 20px;
          i {
            margin-right: 20px;
          }
        }

      }
      .code-list {
        display: flex;
        justify-content: space-between;
        padding: 0 20px;
        .code-tree {
          width: 30%;
          .code-tree-title {
            color: @Acolor1;
            height: 40px;
            margin-bottom: 10px;
            h6, i {
              margin-top: 20px;
            }
          }
          .code-trees {
            height: 390px;
            border: @BD1;
            overflow-x: auto;
          }
        }
        .code-source {
          margin: 20px 0 0 10px;
          width: 69%;
          .code-bottom{
            margin-bottom: 10px;
          }
          .code-source-info {
            height: 390px;
            overflow-x: auto;
          }
        }
      }
    }
    .certifing {
      .top-info {
        border: 0;
      }
      .upload {
        .file {
          width: 10px;
        }
        .err-info {
          width: 450px;
          margin: 30px auto 20px;
          p {
            margin: 20px 0 0 0;
          }
        }
        .el-button {
          width: 350px;
          margin: 40px auto 0;
          background-color: #00DB82;
          border-color: #00DB82;
        }
        .upload-info {
          width: 460px;
          margin: 60px auto;
          border-top: @BD1;
          border-bottom: @BD1;
          padding: 10px 0;
          p {
            padding: 0 0 10px 0;
            line-height: 18px;
          }
        }
      }
      .params {
        .params-img {
          width: 750px;
          height: 145px;
          margin: 30px auto 0;
          .ul-img {
            li {
              width: 33.33%;
              .p_bg {
                width: 60px;
                height: 60px;
                border-radius: 30px;
                line-height: 60px;
                padding: 0 0 0 25px;
                margin: 0 60px;
                background: #f64b3e;
                color: white;
              }
              .p_title {
                margin: 20px auto 0;
              }
              .line {
                border: 1px dashed #f64b3e;
                position: absolute;
                width: 190px;
                margin: -65px 0 0 120px;
              }
            }
          }
        }
        .params-table {
          width: 630px;
          margin: 0 auto 0;
          border: @BD1;
          .tabs_header {
            border-bottom: @BD1;
          }
          .froms {
            margin: 10px 30px 0;
            .froms-btn {
              margin: 10px 0 0 0;
              .el-button {
                width: 200px;
              }
            }
          }
        }
      }
    }
    .loading {
      .el-dialog {
        background-color: transparent !important;
        box-shadow: 0 0 0 rgba(0, 0, 0, .3);
      }
    }
  }
  @media (max-width: 1000px){
    .code-info{
      .certified{
        .code-list{
          flex-wrap: wrap;
          .code-tree{
            width: 100%;
          }
          .code-source{
            width: 100%;
            margin-left: 0;
          }
        }
      }
    }
  }
@media (max-width: 568px){
  .code-info{
    .certifing{
      .upload {
        .upload-info{
          width: initial;
        }
      }
    }
  } 
}
</style>
