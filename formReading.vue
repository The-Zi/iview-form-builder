/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-05-30 17:56:38
 */

<template>
    <Row>
      <!-- 表单基础信息 -->
      <Col class="form-row" span="24">
        <Row>
          <!-- 表单类型编号 -->
          <Col class="form-number-wrap" :xs="24" :sm="24" :md="16" :lg="16">
            <p>单号:&nbsp;&nbsp;<span class="form-number">{{formBase.applyNo}}</span></p>
          </Col>

          <!-- 表单名称 -->
          <Col class="form-type-wrap" :xs="24" :sm="24" :md="24" :lg="24">
            <h1>{{formBase.applyCategoryName}}</h1>
          </Col>

          <!-- 标题 -->
          <Col class="form-title-wrap" :xs="24" :sm="24" :md="16" :lg="16">
            <p>标题：&nbsp;&nbsp;{{formBase.title}}</p>
          </Col>

          <!-- 紧急程度 -->
          <Col class="form-status-wrap" :xs="24" :sm="24" :md="8" :lg="8">
            <p v-if="formBase.emergencyDegree === 0">紧急程度：&nbsp;&nbsp;一般</p>
            <p v-else-if="formBase.emergencyDegree === 1">紧急程度：&nbsp;&nbsp;紧急</p>
          </Col>
        </Row>
      </Col>

      <!-- 表单申请信息 -->
      <Col v-if="pattern === 'user'" class="form-row" span="24">
        <Row>
          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请人：{{formBase.username}}</p>
          </Col>

          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请部门：{{formBase.depName}}</p>
          </Col>

          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请日期：{{formBase.createTime}}</p>
          </Col>
        </Row>
      </Col>

      <!-- 表单构造区域 -->
      <Col class="form-row" span="24">
        <!-- 表单渲染 -->
        <Row class="form-grid-wrap">
          <!-- 表单行 -->
          <Col class="form-grid-row" v-for="(row, rowIndex) in formData" :key="rowIndex" span="24">
            <!-- 表单列 -->
            <Row class="form-grid-col-wrap">
              <Col class="form-grid-col" v-for="(col, colIndex) in row"
               :span="col.offset?setColSize(rowIndex) + (setColSize(rowIndex)) : setColSize(rowIndex)"
                :key="colIndex">
                <div class="form-grid-render-zoon">
                  <!-- 渲染表单组件 -->
                  <div class="form-modules-item" v-for="(modules, mIndex) in col" :key="'form-modules-' +
                   mIndex">
                    <p>
                      {{modules.label}}
                      <br>
                      <!-- 值格式化处理 -->
                      {{
                        handleValue(modules, modules.value)
                      }}
                    </p>
                  </div>
                </div>
              </Col>
            </Row>
          </Col>
        </Row>
      </Col>
    </Row>
</template>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "./styles/baseStyles";
@import "./styles/commonStyles";


  // 表单编号
  .form-number-wrap {
    padding: $padding;
    padding-bottom: 0;
  }
  .form-number {
    text-align: left;
  }

  // 表单类型
  .form-type-wrap {
    height: 100px;
    padding: 0 $padding;
    padding-bottom: 33px;
    line-height: 100px;
    text-align: center;
    font-size: 16px;
    > h1 {
      font-weight: normal;
    }
  }

  // 标题、紧急程度
  .form-status-wrap,
  .form-title-wrap {
    padding: $padding;
    .ivu-form-item {
      margin-bottom: 0;
    }
  }

  // 表单行
  .form-row {
    border: 1px solid #ccc;
    border-bottom: none;
    &:last-child {
      border: 1px solid #ccc;
    }
    // 表单列
    .form-col {
      padding: $padding;
      border-right: 1px solid #ccc;
      &:last-child {
        border-right: none;
      }
    }
  }


  // 表单项渲染区域
 .form-grid-wrap {
   min-height: $colMinH - 70;
   // 表单栅格行
  .form-grid-row {
    border-bottom: 1px solid #ccc;
    &:last-child {
      border-bottom: none;
    }
    // 表单行外包围
   .form-grid-col-wrap {
     display: -webkit-box;
     display: -webkit-flex;
     display: -ms-flexbox;
     display: flex;
     flex-wrap: wrap;
        // 表单栅格列
       .form-grid-col {
         display: flex;
         flex-direction: column;
         padding: $padding;
         min-height: $colMinH - 70;
         // 表格边框
         border-right: 1px solid #ccc;
         &:last-child {
           border-right: none;
         }
          // 表单渲染区
         .form-grid-render-zoon {
           height: 100%;
           .form-grid-render-activer {
             height: 100%;
             border: 1px dashed #adadad;
           }
           .form-grid-item {
             width: 100%;
           }
         }
       }
     }
    }
  }
</style>

<script>
  import formUpload from './formUpload'

  export default {
    // 本组件名
    name: "form-reading",

    // 组件
    components:{
      formUpload
    },

    // 自定义属性
    props: {
      // 基础数据：表单类型、标题、紧急程度、用户名、用户所属部分、填表日期
      baseData: {
        default: ()=>{
          return {
            // 表单号
            applyNo:"",
              //表单种类id
              type:'',
            // 表单名称
            applyCategoryName:"",
            // 标题
            title:"",
            // 紧急状态
            emergencyDegree: null,
            // 用户名
            username:"",
            // 部门名
            depName:"",
            // 提交时间
            createTime:"",
              //表单数据
              jsonContent:[]
          }
        }
      },
      // 表单数据
      renderData: {
        default: Array
      },
      // 表单键值对
      keyValue:{
        default:Object
      },
      // 表单渲染模式
      pattern: {
        default: "user"
      },
    },

    // 数据
    data() {
      return {
        // 表单基础数据
        formBase: {},
        // 表单渲染数据
        formData: [],
        // 表单键值对
        formKeyValue: {},
      }
    },

    // 方法
    methods: {
      // 设置列大小
      setColSize(colIndex) {
        // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
        // 前提是必须在html文件的<head>内添加
        // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
        let deviceWidth =  window.screen.width;
        if (deviceWidth && deviceWidth < 992) {
          return 24
        }

        let col = this.formData[colIndex];
        let collength = this.formData[colIndex].length;
        if(collength === 5) {
            this.formData[colIndex][collength - 1].offset = true;
            return 24 / (collength + 1)
        } else {
            return 24 / collength
        }
      },

      // 设置表单键值
      setFormKeyValue() {
        if (this.formData.length > 0 &&  Object.getOwnPropertyNames(this.formKeyValue).length > 0) {
          // 查找行
          for (let index1 = 0; index1 < this.formData.length; index1++) {
            // 查找列
            for (let index2 = 0; index2 < this.formData[index1].length; index2++) {
              // 查找组件
              for (let index3 = 0; index3 < this.formData[index1][index2].length; index3++) {
                // 设置键值
                for (const key in this.formKeyValue) {
                  if (this.formKeyValue.hasOwnProperty(key)) {
                    if (this.formData[index1][index2][index3].name === key) {
                      this.formData[index1][index2][index3].value = this.formKeyValue[key];
                    }
                  }
                }
              }
            }
          }
        }
      },

      // 值处理
      handleValue(module, value) {
        if (Array.isArray(value)) {
          if (module.type === "时间日期范围") {
            // 检查是不是没有填写时间日期范围
            let valNullNum = 0;
            for (let index = 0; index < value.length; index++) {
              if (value[index] === "") {
                valNullNum = valNullNum + 1;
                if (valNullNum === value.length) {
                  return "未填"
                }
              }
            }

            // 格式化时间日期范围
            let date1 = new Date(value[0]);
            let date2 = new Date(value[1]);
            date1Y = date1.getFullYear() + "年";
            date1M = date1.getMonth() + 1 > 10? date1.getMonth() + 1 + "月": "0"+(date1.getMonth() + 1) + "月";
            date1D = date1.getDate() > 10? date1.getDate() + "日": "0"+ date1.getDate() + "日";
            date1H = date1.getHours() > 10? date1.getHours() + "时": "0"+ date1.getHours() + "时";
            date1Mi = date1.getMinutes() > 10? date1.getMinutes() + "分": "0"+ date1.getMinutes() + "分";
            date1 = date1Y + date1M + date1D + date1H + date1Mi;

            date2Y = date2.getFullYear() + "年";
            date2M = date2.getMonth() + 1 > 10? date2.getMonth() + 1 + "月": "0"+(date2.getMonth() + 1) + "月";
            date2D = date2.getDate() > 10? date2.getDate() + "日": "0"+ date2.getDate() + "日";
            date2H = date2.getHours() > 10? date2.getHours() + "时": "0"+ date2.getHours() + "时";
            date2Mi = date2.getMinutes() > 10? date2.getMinutes() + "分": "0"+ date2.getMinutes() + "分";
            date2 = date2Y + date2M + date2D + date2H + date2Mi;

            return date1 + " - " + date2
          } else {
            for (let index = 0; index < value.length; index++) {
              if (value[index] === "") {
                value.splice(index, 1);
              }
            }
            return value.join("、")
          }
        } else if (module.type === "时间日期") {
            // 格式化时间日期范围
            if (value == "") {
              return "未填写"
            }

            let date1 = new Date(value);
            date1Y = date1.getFullYear() + "年";
            date1M = date1.getMonth() + 1 > 10? date1.getMonth() + 1 + "月": "0"+(date1.getMonth() + 1) + "月";
            date1D = date1.getDate() > 10? date1.getDate() + "日": "0"+ date1.getDate() + "日";
            date1H = date1.getHours() > 10? date1.getHours() + "时": "0"+ date1.getHours() + "时";
            date1Mi = date1.getMinutes() > 10? date1.getMinutes() + "分": "0"+ date1.getMinutes() + "分";
            date1 = date1Y + date1M + date1D + date1H + date1Mi;

            return date1
        } else if (module.type === "预算") {
          if (Number(value) === 0) {
            return "预算内"
          } else if (Number(value) === 1) {
            return "预算外"
          }
        } else {
          return value
        }
      }
    },

    // 生命周期钩子：组件创建前
    created: function(){
      // 设置键值对
      // this.setFormKeyValue();
    },

    // 监测数据变化
    watch: {
      // 设置渲染数据
      renderData: function (newVal, oldVal) {
        this.formData = newVal;
        // 设置键值对
        this.setFormKeyValue();
      },
      // 设置基础数据
      baseData: function (newVal, oldVal) {
        this.formBase = newVal;
      },
      // 设置表单键值
      keyValue: function (newVal, oldVal) {
        this.formKeyValue = newVal;
        this.setFormKeyValue();
      }
    },

  }
</script>
