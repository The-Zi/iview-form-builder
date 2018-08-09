/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 16:34:55
 */


<template>
    <Row>
      <!-- 表单基础信息 -->
      <Col class="iview-form-builder-row" span="24" v-if="nowMode !== module.detailed.type">
        <Row>
          <!-- 表单类型编号 -->
          <Col class="form-number-wrap" span="24">
            <p>单号:&nbsp;&nbsp;<span class="form-number">{{formBase.applyNo}}</span></p>
          </Col>

          <!-- 表单名称 -->
          <Col class="form-type-wrap" span="24">
            <h1>{{formBase.applyCategoryName}}</h1>
          </Col>

          <!-- 标题 -->
          <Col class="form-title-wrap" span="24">
            <p>标题：&nbsp;&nbsp;{{formBase.title}}</p>
          </Col>

          <!-- 紧急程度 -->
          <Col class="form-status-wrap" span="24">
            <p v-if="formBase.emergencyDegree === 0">紧急程度：&nbsp;&nbsp;一般</p>
            <p v-else-if="formBase.emergencyDegree === 1">紧急程度：&nbsp;&nbsp;紧急</p>
          </Col>
        </Row>
      </Col>

      <!-- 表单申请信息 -->
      <Col class="iview-form-builder-row" span="24" v-if="pattern === 'user' &&
      nowMode !== module.detailed.type" >
        <Row>
          <Col class="iview-form-builder-col" span="24">
            <p>申请人：{{formBase.username}}</p>
          </Col>

          <Col class="iview-form-builder-col" span="24">
            <p>申请部门：{{formBase.depName}}</p>
          </Col>

          <Col class="iview-form-builder-col" span="24">
            <p>申请日期：{{formBase.createTime}}</p>
          </Col>
        </Row>
      </Col>

      <!-- 表单构造区域 -->
      <Col class="iview-form-builder-row" span="24">
        <!-- 表单渲染 -->
        <Row class="form-grid-wrap">
          <Form ref="thisForm" :model="formModel" :rules="ruleValidate" label-position="top" inline>
            <!-- 表单行 -->
            <Col class="form-grid-row" v-for="(row, rowIndex) in formData" :key="rowIndex" span="24">
              <!-- 表单列 -->
              <Row class="form-grid-col-wrap">
                <Col class="form-grid-col" v-for="(col, colIndex) in row"
                 :span="col.offset?setColSize(rowIndex) + (setColSize(rowIndex)) : setColSize(rowIndex)"
                  :key="colIndex">
                  <div class="form-grid-render-zoon">
                    <!-- =============== 表单组件渲染 =============== -->
                    <div v-for="(modules, mIndex) in col" :key="'form-modules-' + mIndex">
                      <p class="form-reading-data">
                        <span v-if="modules.require" style="color: red;">*</span>
                        {{ modules.label }}
                        <br>
                        <!-- 值格式化处理 -->
                        {{ handleValue(modules) }}
                      </p>

                      <!-- 递归渲染明细组件 -->
                      <div v-if="modules.type === module.detailed.type">
                        <iview-form-builder-reading
                        :mode="module.detailed.type"
                        :renderData="modules.value.renderData"
                        :keyValue="modules.value.keyValue">
                        </iview-form-builder-reading>
                      </div>
                    </div>

                    <!-- 渲染表单组件：明细 -->
                    <!-- <div v-for="(modules, mIndex) in col" :key="mIndex"> -->
                      <!-- <div v-if="modules.type === module.detailed.type"> -->
                        <!-- 递归渲染明细组件 -->
                        <!-- <iview-form-builder-reading
                        :mode="module.detailed.type"
                        :renderData="modules.value.renderData"
                        :keyValue="modules.value.keyValue"
                        > -->
                                               <!-- :renderData="handleDetailData(modules)" -->
                        <!-- :keyValue="modules.value.keyValue" -->
                        <!-- </iview-form-builder-reading>
                      </div>
                    </div> -->

                  </div>
                </Col>
              </Row>
            </Col>
          </Form>
        </Row>
      </Col>
    </Row>
</template>


<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/common";
@import "../styles/formHeader";
</style>


<script>
// =============== 导入模块 ===============
// 配置文件
import config from '../config';
// 表单组件
import formElements from './elements';

export default {
  // 组件名
  name: "iview-form-builder-reading",

  // 组件
  components:{
    formElements
  },

  // 自定义属性
  props: {
    // 组件模式
    mode: {
      default: config.mode.render
    },

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
            emergencyDegree: 0,
            // 用户名
            username:"",
            // 部门名
            depName:"",
            // 提交时间
            createTime:"",
              //表单数据
              jsonContent:[],
              //上传文件列表
              commFileResourceList:[],
              //表单id
              formbuilderId:''
          }
        }
    },

    // 渲染用数据
    renderData: {
      default: function () {
        return ["22","as"]
      }
    },

    // 表单键值
    keyValue:{
      default: Object
    },

    // 表单渲染模式
    pattern: {
      default: "user"
    },
  },

  // 计算属性
  computed: {
    nowMode() {
      return this.mode
    },
    // 表单基础数据
    formBase() {
      return this.baseData
    },
    // 表单渲染数据
    formData() {
      return this.renderData
    },
    // 表单键值对
    formKeyValue() {
      return this.keyValue
    }
  },

  // 数据
  data() {
    return {
      // 组件模式选项
      modeOption: config.mode,
      // 表单组件配置
      module: config.formElement,
      // 基础数据验证规则
      baseDataRule: {
        title: [{required: true, message: '必填，不能为空', trigger: 'blur'}],
        emergencyDegree: [{type:"number",required: true, message: '必填', trigger: 'blur'}]
      },
      // 临时表单数据,用于表单验证
      formModel: {},
      // 表单验证配置
      ruleValidate: {},
      // 金额最大值
      moneyMax: 999999999999999.9999,
      // 表单组件数量计数器
      modulesLength: {},
      // 表单组件数量唯一下标计数器
      modulesIndex: {},
      // 表单组件-明细，表单数据
      detailedRenderData: {},
      // 可计算对象列表
      calculatorList: {},
      // 计算公式组件列表
      countFormulaList: {},
    }
  },

  // 方法
  methods: {
    // test(params){
    //   return JSON.parse(params)
    // },
    // 获取表单数据模板
    getFormDataTemplate: function (params) {
      switch (params) {
        case "form":
          return [[[]]]
          break;
        case "row":
            return [[]]
          break;
        case "col":
            return []
          break;
        case "el":
            return {}
          break;
        default:
            return [[[]]]
          break;
      }
    },

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
              let target = this.formData[index1][index2][index3];
              // 设置键值
              for (const key in this.formKeyValue) {
                if (this.formKeyValue.hasOwnProperty(key)) {
                  if (target.name === key) {
                    // 对表单组件：明细 的值进行处理
                    if (target.type === this.module.detailed.type) {
                      target.value = {
                        renderData: JSON.parse(this.formKeyValue[key].renderData),
                        keyValue: JSON.parse(this.formKeyValue[key].keyValue)
                      };
                    } else {
                      target.value = this.formKeyValue[key];
                    }
                  }
                }
              }
            }
          }
        }
      }
    },

    // 处理时间日期格式
    hanleDateFormat(params){
      // 日期格式处理
      let formatHanle = (params) => {
        // 年月日时分秒
        let dateY, dateM, dateD, dateH, result;
        let date = new Date(params.value);
        dateY = date.getFullYear();
        dateM = date.getMonth() + 1;
        dateD = date.getDate();
        dateH = date.getHours();
        dateS = date.getMinutes();
        result = dateY + "/" + dateM +  "/"+ dateD + "/" + " " + dateH + ":" + dateS;

        // 日期格式处理
        if (params.format && typeof params.format === "string") {
          // 传递进来的时间日期格式模板
          let formatResult = params.format;
          // 年
          formatResult = formatResult.replace("yyyy", dateY);
          // 月
          if (params.format.search("MM") >= 0) {
            formatResult = formatResult.replace("MM", dateM + 1 >= 10 ? dateM : "0" + (dateM + 1));
          } else if (params.format.search("mm") >= 0) {
            formatResult = formatResult.replace("mm", dateM + 1);
          }
          // 日
          if (params.format.search("DD") >= 0) {
            formatResult = formatResult.replace("DD", dateD >= 10 ? dateD : "0" + dateD);
          } else if (params.format.search("dd") >= 0) {
            formatResult = formatResult.replace("dd", dateD);
          }
          // 时
          if (params.format.search("HH") >= 0) {
            formatResult = formatResult.replace("HH", dateH >= 10 ? dateH : "0" + dateH);
          } else if (params.format.search("hh") >= 0) {
            formatResult = formatResult.replace("hh", dateH);
          }
          // 秒
          if (params.format.search("SS") >= 0) {
            formatResult = formatResult.replace("SS", dateS >= 10 ? dateS : "0" + dateS);
          } else if (params.format.search("ss") >= 0) {
            formatResult = formatResult.replace("ss", dateS);
          }
          return formatResult
        } else {
          return result
        }
      };

      // 日期组件类型判断，范围或单日
      if (Array.isArray(params.value) && params.value.length === 2) {
        if (!params.value[0] || !params.value[1]) {
          return ""
        }
        // ，空的时间日期范围处理不是很完美还需要完善
        let result = "";

        result = formatHanle({value: params.value[0], format: params.format}) + " - ";
        result = result + formatHanle({value: params.value[1], format: params.format});
        return result;
      } else {
        if (!params.value) {
          return ""
        }
        return formatHanle({value: params.value, format: params.format});
      }
    },

    // 值处理
    handleValue(module) {

      // 不处理表单组件：明细的值
      if (module.type === this.module.detailed.type) {
        return
      }

      // 处理数组类型的值
      if (Array.isArray(module.value)) {
        // 处理时间日期范围组件的值
        if (module.type === "时间日期范围" || module.type === this.module.dateTimeRange.type) {
          if (module.format) {
            return this.hanleDateFormat(module);
          } else {
            module.format = "yyyy年MM月DD日";
            return this.hanleDateFormat(module);
          }
        // 处理其它值是数组类型的组件的值
        } else {
          for (let index = 0; index < module.value.length; index++) {
            if (module.value[index] === "") {
              module.value.splice(index, 1);
            }
          }
          return module.value.join("、")
        }
      // 处理单日时间日期组件的值
      } else if (module.type === "时间日期" || module.type === this.module.dateTime.type) {
          if (module.format) {
            return this.hanleDateFormat(module);
          } else {
            module.format = "yyyy年MM月DD日";
            return this.hanleDateFormat(module);
          }
      } else if (module.type === "预算" || module.type === this.module.budget.type) {
        if (Number(module.value) === 0) {
          return "预算内"
        } else if (Number(module.value) === 1) {
          return "预算外"
        }
      } else {
        return module.value
      }
      // // 处理数组类型的值
      // if (Array.isArray(value)) {
      //   // 处理时间日期范围组件的值
      //   if (module.type === "时间日期范围" || module.type === this.module.dateTimeRange.type) {
      //     if (module.format) {
      //       return this.hanleDateFormat(module);
      //     } else {
      //       module.format = "yyyy年MM月DD日";
      //       return this.hanleDateFormat(module);
      //     }
      //   // 处理其它值是数组类型的组件的值
      //   } else {
      //     for (let index = 0; index < value.length; index++) {
      //       if (value[index] === "") {
      //         value.splice(index, 1);
      //       }
      //     }
      //     return value.join("、")
      //   }

      //   if (module.type === "时间日期范围" && Array.isArray(value) && value.length > 0) {
      //     for (let index = 0; index < value.length; index++) {
      //       return this.hanleDateFormat({value:value[index], format: "yyyy年MM月DD日 HH:SS"});
      //     }
      //   } else {
      //     for (let index = 0; index < value.length; index++) {
      //       if (value[index] === "") {
      //         value.splice(index, 1);
      //       }
      //     }
      //     return value.join("、")
      //   }

      // // 处理单日时间日期组件的值
      // } else if (module.type === "时间日期" || module.type === this.module.dateTime.type) {
      //     if (value && typeof value === "number" || typeof value === "string" ) {
      //       if(value) {
      //          return this.hanleDateFormat({value: value, format: "yyyy年MM月DD日 HH:SS"});
      //       } else {
      //         return ""
      //       }
      //     }
      // } else if (module.type === "预算" || this.module.budget.type) {
      //   if (Number(value) === 0) {
      //     return "预算内"
      //   } else if (Number(value) === 1) {
      //     return "预算外"
      //   }
      // } else {
      //   return value
      // }
    },

    // 表单组件-明细 渲染数据处理
    handleDetailData: function (params) {
      // 与每个表单组件-明细相对应的渲染数据
      // if (!this.detailedRenderData[params.id]) {
        // this.detailedRenderData[params.id] = this.getFormDataTemplate();
      // }
      this.detailedRenderData[params.id] = this.getFormDataTemplate();

      // 根据原始模板数据设置渲染数据
      if (params.renderTemplate.length > 0 && this.detailedRenderData[params.id][0][0].length === 0) {
        for (let index = 0; index < params.renderTemplate.length; index++) {
          let temp = Object.assign({}, params.renderTemplate[index]);
          this.detailedRenderData[params.id][0][0].push(temp);
        }
      }
      return this.detailedRenderData[params.id]
    },
  },

  // 监测数据变化
  watch: {
    // 更新表单渲染数据
    renderData: function (newVal, oldVal) {
      // 设置键值对
      this.setFormKeyValue();
    },
    // 更新表单渲染数据
    keyValue: function (newVal, oldVal) {
      // 设置键值对
      this.setFormKeyValue();
    }
  }
}
</script>
