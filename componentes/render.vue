/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 16:41:23
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

          <Form ref="baseDataForm" :model="formBase" :rules="baseDataRule" :label-width="100"
           label-position="left">
          <!-- 标题 -->
          <Col class="form-title-wrap" span="24">
            <FormItem label="标题：" prop="title">
              <Input v-model="formBase.title" placeholder="必填，请对本次申请进行说明" @keyup.native="clearSpace">
              </Input>
            </FormItem>
          </Col>

          <!-- 紧急程度 -->
          <Col class="form-status-wrap" span="24">
            <FormItem label="紧急程度：" prop="emergencyDegree" required>
              <RadioGroup v-model="formBase.emergencyDegree" @on-change="setEmergencyDegree">
                <Radio :label="0">一般</Radio>
                <Radio :label="1">紧急</Radio>
              </RadioGroup>
            </FormItem>
          </Col>
          </Form>
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
          <Form ref="thisForm" :model="formModel" :rules="ruleValidate">
            <!-- 表单行 -->
            <Col class="form-grid-row" v-for="(row, rowIndex) in formData" :key="rowIndex" span="24">
              <!-- 表单列 -->
              <Row class="form-grid-col-wrap">
                <Col class="form-grid-col" v-for="(col, colIndex) in row"
                :span="col.offset?setColSize(rowIndex) + (setColSize(rowIndex)) : setColSize(rowIndex)"
                :key="colIndex">
                  <div class="form-grid-render-zoon">

                    <!-- =============== 表单组件渲染 =============== -->
                    <formElements
                    :mode="modeOption.render"
                    :modulesData="{rowIndex: rowIndex,colIndex:colIndex, col: col}"
                    @moneyFormat="moneyFormat"
                    @count="performCount"
                    @syncEvent="formDataSync"
                    @findRequire="setRequired"
                    @findCountFormula="saveCountFormula"
                    @findCalculator="saveCalculator">
                    </formElements>

                    <!-- 渲染表单组件：明细 -->
                    <div v-for="(modules, mIndex) in col" :key="mIndex">
                      <div v-if="modules.type === module.detailed.type">
                        <FormItem class="iview-form-builder-modules-wrap"
                        v-if="modules.type === module.detailed.type"
                        :required="modules.require"
                        :label="modules.label">

                          <!-- 递归渲染明细组件 -->
                          <iview-form-builder-render
                          :ref="modules.id"
                          :mode="module.detailed.type"
                          :renderData="handleDetailData(modules)"
                          :calculator="calculatorList"
                          :countFormula="countFormulaList"
                          @moneyFormat="moneyFormat"
                          @count="performCount"
                          @syncEvent="formDataSync"
                          @findRequire="setRequired"
                          @findCountFormula="saveCountFormula"
                          @findCalculator="saveCalculator">
                          </iview-form-builder-render>

                          <!-- 增减明细 -->
                          <div class="form-modules-detailed-action">
                              <ButtonGroup>
                                  <Button type="ghost" icon="minus" @click.native="minusDetailModuels(modules)">
                                    {{modules.actionMinus}}
                                  </Button>
                                  <Button type="ghost" icon="plus" @click.native="plusDetailModuels(modules)">
                                    {{modules.actionPlus}}
                                  </Button>
                              </ButtonGroup>
                          </div>
                        </FormItem>
                      </div>
                    </div>

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
import detailed from './detailed';

export default {
  // 本组件名
  name: "iview-form-builder-render",

  // 组件
  components:{
    formElements,
    detailed
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
      default: Array
    },

    // 表单键值
    keyValue:{
      default: Object
    },

    // 表单渲染模式
    pattern: {
      default: "user"
    },

    // 计算对象
    calculator: {
      default: Object
    },

    // 计算公式
    countFormula: {
      default: Object
    }
  },

  // 计算属性
  computed: {
    nowMode: function () {
      return this.mode
    },
    // 可计算对象列表
    calculatorList: {
      get: function () {
        return this.calculator
      },
      set: function (newValue) {
        this.calculator = Object.assign({}, this.calculator, newValue);
      }
    },
    // 计算公式组件列表
    countFormulaList: {
      get: function () {
        return this.countFormula
      },
      set: function (newValue) {
        this.countFormula = Object.assign({}, this.countFormula, newValue);
      }
    },
  },

  // 数据
  data() {
    return {
      // 组件模式选项
      modeOption: config.mode,
      // 表单组件配置
      module: config.formElement,
      // 表单基础数据
      formBase: {},
      // 基础数据验证规则
      baseDataRule: {
        title: [{required: true, message: '必填，不能为空', trigger: 'blur'}],
        emergencyDegree: [{type:"number",required: true, message: '必填', trigger: 'blur'}]
      },
      // 表单键值对
      formKeyValue: {},
      // 表单渲染数据
      formData: [],
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
      // // 可计算对象列表
      // calculatorList: {},
      // // 计算公式组件列表
      // countFormulaList: {},
      // 保存来自明细组件内的计算公式
      detailedCountFoumula: {},
    }
  },

  // 方法
  methods: {
    // 深克隆
    deepClone(obj) {
      let result;
      //返回传递给他的任意对象的类
      let isClass = (o)=>{
        if(o === null) return "Null";
        if(o === undefined) return "Undefined";
        return Object.prototype.toString.call(o).slice(8,-1);
      }
      let oClass = isClass(obj);
      //确定result的类型
      if(oClass === "Object"){
          result = {};
      }else if(oClass === "Array"){
          result=[];
      }else{
          return obj;
      }
      for (let key in obj) {
          let copy = obj[key];
          if(isClass(copy) == "Object"){
            result[key] = arguments.callee(copy);//递归调用
          }else if(isClass(copy) == "Array"){
            result[key] = arguments.callee(copy);
          }else{
            result[key] = obj[key];
          }
      }
      return result
    },

    // 清除空格
    clearSpace() {
      this.formBase.title = this.formBase.title.replace(/(^[\s\n\t]+|[\s\n\t]|[\s\n\t]+$)/g, "");
    },

    // 获取表单数据模板
    getFormDataTemplate(params) {
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

    // 设置列大小，使用iview的栅格系统进行布局
    setColSize(colIndex) {
        // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
        // 前提是必须在html文件的<head>内添加
        // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
        let deviceWidth =  window.screen.width;
        if (deviceWidth && deviceWidth < 992) {
          return 24
        }

        // 根据列数量设置列宽度
        let col = this.formData[colIndex];
        let collength = this.formData[colIndex].length;
        if(collength === 5) {
            this.formData[colIndex][collength - 1].offset = true;
            return 24 / (collength + 1)
        } else {
            return 24 / collength
        }
    },

    // 表单数据同步
    formDataSync() {
      // 当前渲染的表单数据，包括结构和值
      let formData = this.formData;

      for (let index1 = 0; index1 < formData.length; index1++) {
        for (let index2 = 0; index2 < formData[index1].length; index2++) {
          for (let index3 = 0; index3 < formData[index1][index2].length; index3++) {
            // 目标组件
            let target = formData[index1][index2][index3];
            let value = target.value;

            // 验证必填项用数据
            this.formModel[target.name] = value;
            // 对时间日期范围组件的空值做处理，以实现必填项判断为未填
            if (Array.isArray(value) && value.length === 2 && value[0] === "" && value[1] === "") {
              this.formModel[target.name] = "";
            }
          }
        }
      }
    },

    // 取出表单的键值对
    getKeyValue(params = ""){
      // 必填项状态
      let status = true;

      // 遍历表单结构，把所有表单元素的 key 和 value 取出来
      for (let index1 = 0; index1 < this.formData.length; index1++) {
        for (let index2 = 0; index2 < this.formData[index1].length; index2++) {
          for (let index3 = 0; index3 < this.formData[index1][index2].length; index3++) {
            let targetModule = this.formData[index1][index2][index3];
            let moduleType = targetModule.type;
            let moduleValue = targetModule.value;
            let moduleName = targetModule.name;

            // 将时间日期组件的value转成时间戳
            if (moduleType === "时间日期" || moduleType === "时间日期范围" ||
            moduleType === this.module.dateTime.type || moduleType === this.module.dateTimeRange.type) {

              // 时间日期范围
              if (Array.isArray(moduleValue)) {
                this.formKeyValue[moduleName] = [];
                for (let index = 0; index < moduleValue.length; index++) {
                  if (moduleValue[index] instanceof Date) {
                    this.formKeyValue[moduleName][index] = moduleValue[index].getTime();
                  } else {
                    this.formKeyValue[moduleName][index] = "";
                  }
                }

              // 时间日期
              } else {
                if (moduleValue instanceof Date) {
                  this.formKeyValue[moduleName] = moduleValue.getTime();
                } else {
                  this.formKeyValue[moduleName] = "";
                }
              }

            // 表单组件：明细
            } else if (moduleType === this.module.detailed.type) {

              // 检查当前明细组件内的必填项
              this.$refs[targetModule.id][0].checkRequired((result)=>{
                // 如果必填项都填写了，就将该明细组件对应的表单域的值设为
                // 改明细组件的表单结构和键值对
                if (result) {
                  this.formKeyValue[moduleName] = {
                    // renderData: this.detailedRenderData[targetModule.id],
                    // keyValue: this.$refs[targetModule.id][0].getKeyValue().result
                    renderData: JSON.stringify(this.detailedRenderData[targetModule.id]),
                    keyValue: JSON.stringify(this.$refs[targetModule.id][0].getKeyValue().result)
                  };
                } else {
                  status = result;
                }
              });

            // 其它组件的值都直接保存
            } else {
              this.formKeyValue[moduleName] = moduleValue;
            }
          }
        }
      }

      // 是否过滤表单值
      if (params === "all") {
        return { status: status, result: this.formKeyValue }
      } else {
        // 过滤结果
        let result = {};
        // 过滤表单值
        for (const key in this.formKeyValue) {
          if (this.formKeyValue.hasOwnProperty(key)) {
            // 过滤空字符串和空对象
            if (this.formKeyValue[key] !== "" && this.formKeyValue[key] !== null) {
              result[key] = this.formKeyValue[key];
            // 过滤空数组
            } else if (Array.isArray(this.formKeyValue[key]) && this.formKeyValue[key].length > 0) {
              result[key] = this.formKeyValue[key];
            }
          }
        }
        // 返回过滤结果
        return {status: status, result: result}
      }
    },

    // 处理提交
    handleSubmit() {
      // 检查结果
      let checkResult = 0;
      // 表单数量
      let formNum = 0;

      // 计算iview表单的数量
      // 表单组件: 明细
      for (const key in this.detailedRenderData) {
        if (this.detailedRenderData.hasOwnProperty(key)) {
          formNum += 1;
        }
      }

      // 发现基础数据表单
      if (this.$refs.baseDataForm) {
        formNum += 1;
      }

      // 发现渲染表单
      if (this.$refs.thisForm) {
        formNum += 1;
      }

      // 检查必填项前，对表单数据进行同步，将表单组件的值同步到用于验证必填项的表单组件对应的变量
      this.formDataSync();

      // 基础数据表单必填项检查
      if (this.nowMode === this.modeOption.render || this.$refs.baseDataForm) {
        this.$refs.baseDataForm.validate((result) => {
          if (result) {
            checkResult += 1;
          }
        });
      }

      // 渲染表单必填项检查
      this.$refs.thisForm.validate((result) => {
        if (result) {
          checkResult += 1;
        }
      });

      // 表单组件: 明细，表单必填项检查
      for (const key in this.detailedRenderData) {
        if (this.detailedRenderData.hasOwnProperty(key)) {
          this.$refs[key][0].checkRequired((result)=>{
            if (result) {
              checkResult += 1;
            }
          });
        }
      }

      // 检查所有表单的必填项是否都已填写
      if (checkResult === formNum) {
        // 获取表单键值对
        let keyValue = this.getKeyValue();

        // 表单组件：明细 内的表单结构数据和键值对获取结果状态
        if (keyValue.status === true) {
          // 结果数据
          let result =  { baseData: this.baseData, formData: this.formData, keyValue: keyValue.result };

          // 传递表单基础数据、表单结构数据、表单键值对
          this.$emit("submitEvent", result);
          return result
        }
      } else {
        this.$Modal.error({
          title: "错误",
          content: "请检查必填项"
        });
      }

      // 必填项检查和传递数据
      // if (this.nowMode === this.modeOption.render) {
      //   // 基础数据表单必填项检查
      //   this.$refs.baseDataForm.validate((result) => {
      //     if (result) {
      //       checkResult += 1;
      //     }
      //   });

      //   // 渲染表单必填项检查
      //   this.$refs.thisForm.validate((result) => {
      //     if (result) {
      //       checkResult += 1;
      //     }
      //   });

      //   // 表单组件: 明细，表单必填项检查
      //   for (const key in this.detailedRenderData) {
      //     if (this.detailedRenderData.hasOwnProperty(key)) {
      //       this.$refs[key][0].checkRequired((result)=>{
      //         if (result) {
      //           checkResult += 1;
      //         }
      //       });
      //     }
      //   }

      //   // 检查所有表单的必填项是否都已填写
      //   if (checkResult === formNum) {
      //     // 获取表单键值对
      //     let keyValue = this.getKeyValue();

      //     if (keyValue.status === true) {
      //       // 结果数据
      //       let result =  { baseData: this.baseData, formData: this.formData, keyValue: keyValue.result };

      //       // 传递表单基础数据、表单结构数据、表单键值对
      //       this.$emit("submitEvent", result);

      //       return result
      //     }
      //   } else {
      //     this.$Modal.error({
      //       title: "错误",
      //       content: "请检查必填项"
      //     });
      //   }
      //   // this.$refs.baseDataForm.validate((result) => {
      //   //   if (result) {
      //   //     this.$refs.thisForm.validate((statis)=>{
      //   //       if (statis) {
      //   //         // 获取表单键值对
      //   //         let keyValue = this.getKeyValue();
      //   //         if (keyValue.status === true) {
      //   //           // 结果数据
      //   //           let result =  { baseData: this.baseData, formData: this.formData,
      //   //           keyValue: keyValue.result };

      //   //           // 传递表单基础数据、表单结构数据、表单键值对
      //   //           this.$emit("submitEvent", result);

      //   //           return result
      //   //         }
      //   //       } else {
      //   //         this.$Modal.error({
      //   //           title: "错误",
      //   //           content: "请检查必填项"
      //   //         });
      //   //       }
      //   //     });
      //   //   } else {
      //   //     this.$Modal.error({
      //   //       title: "错误",
      //   //       content: "请检查必填项"
      //   //     });
      //   //   }
      //   // });
      // } else if (this.nowMode === this.module.detailed.type) {
      //   this.$refs.thisForm.validate((statis)=>{
      //     if (statis) {
      //       // 获取表单键值对
      //       let keyValue = this.getKeyValue();
      //       if (keyValue.status === true) {
      //         // 结果数据
      //         let result =  { formData: this.formData, keyValue: keyValue.result };

      //         // 传递表单基础数据、表单结构数据、表单键值对
      //         this.$emit("submitEvent", result);

      //         return result
      //       }
      //     } else {
      //       this.$Modal.error({
      //         title: "错误",
      //         content: "请检查必填项"
      //       });
      //     }
      //   });
      // }
    },

    // 设置必填项
    setRequired(params){
      // 默认为空，未填
      this.formModel[params.name] = "";
      // 验证规则：数据类型
      let type = typeof params.value;
      // 验证规则：验证触发时机
      let trigger = "blur";

      // // 对象类型数据的验证规则
      // if (typeof params.value === "object") {
      //   // 日期类型
      //   if (params.value instanceof Date) {
      //       console.log(params.type)
      //       console.log(params.value)
      //     type = "date";
      //     trigger = "change";
      //   }

      //   // 数组类型
      //   if (Array.isArray(params.value)) {
      //     console.log(params.type)
      //     console.log(params.value)

      //     type = "array";
      //     trigger = "change";
      //   }
      // }

      // if (type === "number") {
      //   trigger = "change";
      // }

      // this.ruleValidate[params.name] = [{required: params.require, message:"必填",
      // type: type, trigger: trigger}];

      // 数字类型必填项：数字输入框类、金额类
      if (params.type === "数字输入框" || params.type  === this.module.inputNumber.type ||
      params.type === this.module.money.type) {

        type = "number";
        trigger = "change";

      // 日期类型必填项：时间日期
      } else if(params.type === "时间日期" || params.type === this.module.dateTime.type) {

        type = "date";
        trigger = "change";

      // 数组类型必填项：时间日期范围、多选框
      } else if(params.type === "时间日期范围" || params.type === this.module.dateTimeRange.type ||
      params.type === "多选" || params.type === this.module.checkbox.type) {
        type = "array";
        trigger = "change";
      }
      // else {
        // this.ruleValidate[params.name] = [{required: params.require, message:"必填",
        // type:"string",trigger: "blur"}];
      // }

      // 设置验证规则
      this.ruleValidate[params.name] = [{required: params.require, message:"必填",
      type: type, trigger: trigger}];
    },

    // 检查必填项
    checkRequired(fn) {
      // 同步验证数据
      this.formDataSync();
      this.$refs.thisForm.validate(fn);
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
                  let target = this.formData[index1][index2][index3];
                  // 找到和键值对匹配的表单元素
                  if (target.name === key) {
                    // 对时间日期组件value做转换处理
                    if (target.type === "时间日期" || target.type === "时间日期范围") {
                      if (Array.isArray(this.formKeyValue[key]) && this.formKeyValue[key].length > 0) {
                        this.formData[index1][index2][index3].value = [];
                        for (let index = 0; index < this.formKeyValue[key].length; index++) {
                          if (this.formKeyValue[key][index]) {
                            let date = new Date(this.formKeyValue[key][index]);
                            this.formData[index1][index2][index3].value[index] = date;
                          }
                        }
                      } else {
                        if (this.formKeyValue[key]) {
                          let date = new Date(this.formKeyValue[key]);
                          this.formData[index1][index2][index3].value = date;
                        }
                      }
                    // 设置表单元素的值
                    } else {
                      this.formData[index1][index2][index3].value = this.formKeyValue[key];
                    }
                  }
                }
              }
            }
          }
        }
      }
    },

    // 设置表单紧急程度
    setEmergencyDegree(val){
      this.formBase.emergencyDegree = Number(val);
    },

    // 重置表单验证状态
    resetFields() {
      // 以表单明细组件模式运行时，不执行对基础数据表单的重置
      if (this.nowMode !== this.module.detailed.type) {
        this.$refs.baseDataForm.resetFields();
      }

      this.$refs.thisForm.resetFields();
    },

    // 保存可计算对象
    saveCalculator: function (params) {
      // 如果当前添加的表单组件是：数字输入框或金额
      // 则同时保存到可计算对象（calculatorList）并 触发自定义事件传递到父组件
      if (params.type === this.module.inputNumber.type || params.type === this.module.money.type ||
      params.type === this.module.count.type) {
        this.calculatorList[params.id] = params;
        // 自定义事件: 添加可计算事件
        this.$emit("findCalculator", params);
      }
    },

    // 保存发现的计算公式组件
    saveCountFormula: function (params) {
      // 保存计算公式组件
      this.countFormulaList[params.id] = params;
      // 自定义事件：发现计算公式组件
      this.$emit("findCountFormula", this.countFormulaList[params.id]);
    },

    // 执行计算
    performCount: function (params, source = "") {
      // 计算明细内的可计算对象
      this.$emit("count", params, source);

      // 计算公式处理
      for (const key1 in this.countFormulaList) {
        if (this.countFormulaList.hasOwnProperty(key1)) {
          // 拥有计算公式的情况下才执行计算操作
          if (this.countFormulaList[key1].countFormulas) {
            // 目标计算公式组件
            let targetFormula = this.countFormulaList[key1];
            // 修改后的计算公式结果
            let formulaResult = "";

            // 可计算对象
            for (const key2 in this.calculatorList) {

              if (this.calculatorList.hasOwnProperty(key2)) {
                // 需要替换进去的可计算对象
                let targetCalculator = this.calculatorList[key2];
                // 将计算用计算公式中，每个可计算对象的id，替换为它们对应的可计算对象的值
                let rules = new RegExp(targetCalculator.id,"g");

                // 对来自表单组件：明细，的计算对象的id进行处理，以获取其在计算公式配置中对应的id
                if (source === this.modeOption.detailed && targetCalculator.id === params.id) {
                  let targetID = params.id.split("-")[0] + "-" + params.id.split("-")[1];
                  rules = new RegExp(targetID,"g");
                }

                // 将值替换进计算公式中对应的位置
                if (formulaResult) {
                  formulaResult = formulaResult.replace(rules, targetCalculator.value);
                } else {
                  formulaResult = targetFormula.countFormulas.replace(rules, targetCalculator.value);
                }

                // 替换计算公式结果中的乘除计算符号
                formulaResult = formulaResult.replace(/[×]/, "*");
                formulaResult = formulaResult.replace(/[÷]/, "/");
              }
            }
            // 执行计算
            try {
              // 根据计算公式计算结果
              let countResult = eval(formulaResult.toString().split("&nbsp;").join(""));
              // 如果计算对象来自表单组件：明细,并且之前执行的计算不包括该计算对象，则新的计算结果与旧的计算结果相加
              if (source === this.modeOption.detailed && !this.detailedCountFoumula[params.id]) {
                targetFormula.value = Number(targetFormula.value) +
                Number(countResult.toFixed(targetFormula.rounding));

                // // 保存当前计算对象用于下次计算时判断是否其来源以及是否需要与旧结果相加
                this.detailedCountFoumula[params.id] = {
                  id: params.id,
                  value: typeof params.value === "null" ? 0 : params.value
                };

              // 普通情况下的计算 与 来自于表单组件：明细，但之前执行的计算包括该计算对象
              } else {
                targetFormula.value = Number(countResult.toFixed(targetFormula.rounding));
              }

              // 将计算结果转换成中文大写数字金额
              this.moneyFormat(targetFormula)
            } catch (error) {
                console.log(error)
            }
          }
        }
      }
    },

    // 货币格式
    moneyFormat: function (params) {
      let money = params.value;
      //汉字的数字
      let cnNums = new Array('零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖');
      //基本单位
      let cnIntRadice = new Array('', '拾', '佰', '仟');
      //对应整数部分扩展单位
      let cnIntUnits = new Array('', '万', '亿', '兆');
      //对应小数部分单位
      let cnDecUnits = new Array('角', '分', '毫', '厘');
      //整数金额时后面跟的字符
      let cnInteger = '';
      // let cnInteger = '整';
      //整型完以后的单位
      // let cnIntLast = params.unit;
      let cnIntLast = '元';
      //最大处理的数字
      let maxNum = this.moneyMax;
      //金额整数部分
      let integerNum;
      //金额小数部分
      let decimalNum;
      //输出的中文金额字符串
      let chineseStr = '';
      //分离金额后用的数组，预定义
      let parts;

      if (money == '') {
        params.cnNum = "";
      }

      money = parseFloat(money);

      //超出最大处理数字
      if (money >= maxNum) {
        params.cnNum = "超出最大可处理数字：999999999999999.9999";
      }

      if (money == 0) {
        chineseStr = cnNums[0] + cnIntLast + cnInteger;
        params.cnNum = chineseStr;
      }

      //转换为字符串
      money = money.toString();

      if (money.indexOf('.') == -1) {
        integerNum = money;
        decimalNum = '';
      } else {
        parts = money.split('.');
        integerNum = parts[0];
        decimalNum = parts[1].substr(0, 4);
      }

      //获取整型部分转换
      if (parseInt(integerNum, 10) > 0) {
        var zeroCount = 0;
        var IntLen = integerNum.length;
        for (var i = 0; i < IntLen; i++) {
          var n = integerNum.substr(i, 1);
          var p = IntLen - i - 1;
          var q = p / 4;
          var m = p % 4;

          if (n == '0') {
            zeroCount++;
          } else {
            if (zeroCount > 0) {
              chineseStr += cnNums[0];
            }
            //归零
            zeroCount = 0;
            chineseStr += cnNums[parseInt(n)] + cnIntRadice[m];
          }

          if (m == 0 && zeroCount < 4) {
            chineseStr += cnIntUnits[q];
          }
        }
        chineseStr += cnIntLast;
      }

      //小数部分
      if (decimalNum != '') {
        var decLen = decimalNum.length;
        for (var i = 0; i < decLen; i++) {
          var n = decimalNum.substr(i, 1);
          if (n != '0') {
            chineseStr += cnNums[Number(n)] + cnDecUnits[i];
          }
        }
      }

      if (chineseStr == '') {
        chineseStr += cnNums[0] + cnIntLast + cnInteger;
      } else if (decimalNum == '') {
        chineseStr += cnInteger;
      }

      // 设置转换结果
      params.cnNum = chineseStr;

      if (money < 0) {
        let x = {
          value: Math.abs(money)
        };
        this.moneyFormat(x)
        params.cnNum = "负" +  this.moneyFormat(x);
      }

      // 返回转换结果
      return chineseStr
    },

    // 表单组件-明细 自增明细内原始模板组件
    plusDetailModuels: function (params) {
      // 只有在表单构建器的渲染模式下，才能根据明细的原始模板数据自增表单组件
      if (this.nowMode === this.modeOption.render) {
        // 旧的渲染数据
        let detailedRenderData = this.detailedRenderData[params.id];
        // 新的渲染数据
        let newData = [];
        // 取出表单组件-明细内组件的模板数据，并修改它们的id和name,防止重复
        for (let index = 0; index < params.renderTemplate.length; index++) {
          // 临时数据
          let temp = this.deepClone(params.renderTemplate[index]);
          // 在id和name属性后新增一个唯一的随机数，以防重复
          let random1 = Math.random();
          let random2 = Math.random();
          if (random1 === random2) {
            random1 = random1 - 0.01;
          }
          temp.id = temp.id + "-" + random1.toString().split(".")[1];
          temp.name = temp.name + "-" + random2.toString().split(".")[1];
          // 将处理好的表单组件数据添加到新的表单数据中
          newData.push(temp);
        }
        // 根据表单组件-明细的渲染模板，新增一组表单组件
        detailedRenderData.push([newData]);
      }
    },

    // 表单组件-明细 自减明细内原始模板组件
    minusDetailModuels: function (params) {
      if (this.nowMode === this.modeOption.render) {
        // 已有或默认的渲染数据
        let target = this.detailedRenderData[params.id];
        // 保留最后一组组件,作为原始数据,不完成删完
        if (target.length > 1) {
          target.splice(target.length - 1, 1);
        }
      }
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

  // 生命周期钩子：组件渲染完毕
  mounted:function () {
    // 重置表单验证状态
    this.resetFields();
  },

  // 监测数据变化
  watch: {
    // 设置基础数据
    baseData: function (newVal, oldVal) {
      this.formBase = newVal;
    },
    // 设置渲染数据
    renderData: function (newVal, oldVal) {
      this.formData = newVal;
    },
    // 设置键值对
    keyValue:function(newVal, oldVal){
      this.formKeyValue = newVal;
      this.setFormKeyValue();
    }
  }
}
</script>
