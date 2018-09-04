/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-08-27 11:39:04
 */


<template>
    <Row>
      <!-- 表单构造区域 -->
      <Col class="iview-form-builder-row" span="24">
        <!-- 表单渲染 -->
        <Row class="form-grid-wrap">
          <Form ref="thisForm" :model="formModel" :rules="ruleValidate" label-position="top">
            <!-- 表单行 -->
            <Col class="form-grid-row" v-for="(row, rowIndex) in formData" :key="rowIndex" span="24">
              <!-- 表单组件-明细模式下删除一组明细 -->
              <Icon class="form-grid-row-del" type="close" v-if="nowMode === modeOption.detailed && rowIndex > 0"
              @click.native="minusDetailModules(row, rowIndex)"></Icon>

              <!-- 表单列 -->
              <Row class="form-grid-col-wrap">
                <Col class="form-grid-col" v-for="(col, colIndex) in row"
                :span="col.offset?setColSize(rowIndex) + (setColSize(rowIndex)) : setColSize(rowIndex)"
                :key="colIndex">
                  <div class="form-grid-render-zoon">

                    <!-- =============== 表单组件渲染 =============== -->
                    <formElements
                    :mode="modeOption.renderDetailed"
                    :modulesData="{rowIndex: rowIndex,colIndex:colIndex, col: col}"
                    :rowIndex="rowIndex"
                    @moneyFormat="moneyFormat"
                    @count="performCount"
                    @syncEvent="formDataSync"
                    @findRequire="setRequiredRules"
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
                          <!-- :modulesData="modules.value.modulesData" -->
                          <!-- :modulesData="handleDetailModulesData(modules)" -->

                          <!-- 递归渲染明细组件 -->
                          <iview-form-builder-render
                          :ref="modules.id"
                          :mode="modeOption.detailed"
                          :calculator="calculatorList"
                          :countFormula="countFormulaList"
                          :renderData="handleDetailRenderData(modules)"
                          @moneyFormat="moneyFormat"
                          @count="performCount"
                          @deleteDetailModules="performCount"
                          @syncEvent="formDataSync"
                          @findRequire="setRequiredRules"
                          @findCountFormula="saveCountFormula"
                          @findCalculator="saveCalculator">
                          </iview-form-builder-render>

                          <!-- 增减明细 -->
                          <div class="form-modules-detailed-action">
                            <Button type="primary" icon="plus" long @click.native="plusDetailModules(modules)">
                              {{modules.actionPlus}}
                            </Button>
                              <!-- <ButtonGroup>
                                  <Button type="ghost" icon="minus" @click.native="minusDetailModules(modules)">
                                    {{modules.actionMinus}}
                                  </Button>
                                  <Button type="ghost" icon="plus" @click.native="plusDetailModules(modules)">
                                    {{modules.actionPlus}}
                                  </Button>
                              </ButtonGroup> -->
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
import config from "../config";
// 表单组件
import formElements from "./elements";
import detailed from "./detailed";

export default {
  // 本组件名
  name: "iview-form-builder-render",

  // 组件
  components: {
    formElements,
    detailed
  },

  // 自定义属性
  props: {
    // 组件模式
    mode: {
      default: config.mode.render
    },

    // 渲染用数据
    renderData: {
      default: Array
    },

    // 表单键值
    modulesData: {
      default: Array
    }
  },

  // 计算属性
  computed: {
    // 当前模式
    nowMode: function() {
      return this.mode;
    }
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
      formModuleData: [],
      // 表单渲染数据
      formData: [[[]]],
      // 临时表单数据,用于表单验证
      formModel: {},
      // 表单验证配置
      ruleValidate: {},
      // 金额最大值
      moneyMax: 999999999999999.9999,
      // 表单组件-明细，表单数据
      detailedRenderData: {},
      // 可计算对象列表
      calculatorList: {},
      // 计算公式组件列表
      countFormulaList: {},
      // 处理完成可执行计算的计算公式
      handleCountFormula: {}
    };
  },

  // 方法
  methods: {
    // 深克隆
    deepClone(obj) {
      let result;
      //返回传递给他的任意对象的类
      let isClass = o => {
        if (o === null) return "Null";
        if (o === undefined) return "Undefined";
        return Object.prototype.toString.call(o).slice(8, -1);
      };
      let oClass = isClass(obj);
      //确定result的类型
      if (oClass === "Object") {
        result = {};
      } else if (oClass === "Array") {
        result = [];
      } else {
        return obj;
      }
      for (let key in obj) {
        let copy = obj[key];
        if (isClass(copy) == "Object") {
          result[key] = arguments.callee(copy); //递归调用
        } else if (isClass(copy) == "Array") {
          result[key] = arguments.callee(copy);
        } else {
          result[key] = obj[key];
        }
      }
      return result;
    },

    // 有效数组检查
    trueArrCheck: function(params) {
      // 判断是否有效的数组值
      if (Array.isArray(params)) {
        // 有效值的数量
        let trueVal = 0;

        // 判断数组内元素是否全部是有效值
        for (let index = 0; index < params.length; index++) {
          if (params[index]) {
            trueVal += 1;
          }
        }

        // 只有在全部元素都有效的情况下，才判为有效值
        if (trueVal === params.length) {
          return params
        // 无效值，设为空数组，触发表单验证规则
        } else {
          return [];
        }
      } else {
        return params;
      }
    },

    // 清除空格
    clearSpace() {
      this.formBase.title = this.formBase.title.replace(
        /(^[\s\n\t]+|[\s\n\t]|[\s\n\t]+$)/g,
        ""
      );
    },

    // 获取表单数据模板
    getFormDataTemplate(params) {
      switch (params) {
        case "form":
          return [[[]]];
          break;
        case "row":
          return [[]];
          break;
        case "col":
          return [];
          break;
        case "el":
          return {};
          break;
        default:
          return [[[]]];
          break;
      }
    },

    // 设置列大小，使用iview的栅格系统进行布局
    setColSize(colIndex) {
      // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
      // 前提是必须在html文件的<head>内添加
      // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
      let deviceWidth = window.screen.width;
      if (deviceWidth && deviceWidth < 992) {
        return 24;
      }

      // 根据列数量设置列宽度
      let col = this.formData[colIndex];
      let collength = this.formData[colIndex].length;
      if (collength === 5) {
        this.formData[colIndex][collength - 1].offset = true;
        return 24 / (collength + 1);
      } else {
        return 24 / collength;
      }
    },

    // 表单数据同步
    formDataSync(action="") {
      // 当前渲染的表单数据，包括结构和值
      let formData = this.formData;

      for (let index1 = 0; index1 < formData.length; index1++) {
        for (let index2 = 0; index2 < formData[index1].length; index2++) {
          for (
            let index3 = 0;
            index3 < formData[index1][index2].length;
            index3++
          ) {
            // 目标组件
            let target = formData[index1][index2][index3];
            let value = target.value;

            this.formModel[target.name] = this.trueArrCheck(value);
            // // 判断是否有效的数组值
            // if (Array.isArray(value)) {
            //   // 有效值的数量
            //   let trueVal = 0;

            //   // 判断数组内元素是否全部是有效值
            //   for (let index = 0; index < value.length; index++) {
            //     if (value[index]) {
            //       trueVal += 1;
            //     }
            //   }

            //   // 只有在全部元素都有效的情况下，才判为有效值
            //   if (trueVal === value.length) {
            //     this.formModel[target.name] = value

            //   // 无效值，设为空数组，触发表单验证规则
            //   } else {
            //     this.formModel[target.name] = [];
            //   }
            // } else {
            //   this.formModel[target.name] = value
            // }
            // 对时间日期范围组件的空值做处理，以实现必填项判断为未填
            // if (
            //   Array.isArray(value) &&
            //   value.length === 2 &&
            //   value[0] === "" &&
            //   value[1] === ""
            // ) {
            //   this.formModel[target.name] = "";
            // }
          }
        }
      }

      // 传递组件数据
      this.$emit("on-change", this.getModulesData(), action);
    },

    // 取出表单组件的数据
    getModulesData(params = "") {
      // 必填项状态
      let status = true;
      // 表单组件数据
      let modulesData = [];

      // 遍历表单结构，把所有表单元素的 key 和 value 取出来
      for (let index1 = 0; index1 < this.formData.length; index1++) {
        for (let index2 = 0; index2 < this.formData[index1].length; index2++) {
          for (
            let index3 = 0;
            index3 < this.formData[index1][index2].length;
            index3++
          ) {
            let targetModule = this.formData[index1][index2][index3];
            let moduleId = targetModule.id;
            let moduleLabel = targetModule.label;
            let moduleName = targetModule.name;
            let moduleValue = targetModule.value;
            let moduleRequire = targetModule.require;
            let moduleType = targetModule.type;

            // 将时间日期组件的value转成时间戳
            if (
              moduleType === this.module.dateTime.type ||
              moduleType === this.module.dateTimeRange.type
            ) {
              // 时间日期范围
              if (Array.isArray(moduleValue)) {
                let temp = {
                  id: moduleId,
                  type: moduleType,
                  label: moduleLabel,
                  name: moduleName,
                  value: [],
                  format: targetModule.format,
                  require: moduleRequire
                };

                for (let index = 0; index < moduleValue.length; index++) {
                  if (moduleValue[index] instanceof Date) {
                    temp.value[index] = moduleValue[index].getTime();
                  } else {
                    temp.value[index] = "";
                  }
                }

                // 添加结果
                modulesData.push(temp);

                // 时间日期
              } else {
                if (moduleValue instanceof Date) {
                  modulesData.push({
                    id: moduleId,
                    type: moduleType,
                    label: moduleLabel,
                    name: moduleName,
                    value: moduleValue.getTime(),
                    format: targetModule.format,
                    require: moduleRequire
                  });
                } else {
                  modulesData.push({
                    id: moduleId,
                    type: moduleType,
                    label: moduleLabel,
                    name: moduleName,
                    value: "",
                    format: targetModule.format,
                    require: moduleRequire
                  });
                }
              }

              // 表单组件：明细
            } else if (moduleType === this.module.detailed.type) {
              // 检查当前明细组件内的必填项
              this.$refs[targetModule.id][0].checkRequired(result => {
                // 如果必填项都填写了，就将该明细组件对应的表单域的值设为
                // 改明细组件的表单结构和键值对
                if (result) {
                  modulesData.push({
                    id: moduleId,
                    type: moduleType,
                    label: moduleLabel,
                    name: moduleName,
                    // 设置表单明细组件的值
                    value: JSON.stringify({
                      // 明细组件的表单渲染数据
                      renderData: JSON.stringify(
                        this.detailedRenderData[targetModule.id]
                      ),
                      // 明细组件的表单组件数据
                      modulesData: JSON.stringify(
                        this.$refs[targetModule.id][0].getModulesData().result
                      )
                    }),
                    require: moduleRequire
                  });

                  // modulesData[moduleName] = {
                  //   renderData: JSON.stringify(
                  //     this.detailedRenderData[targetModule.id]
                  //   ),
                  //   modulesData: JSON.stringify(
                  //     this.$refs[targetModule.id][0].getModulesData().result
                  //   )
                  // };
                } else {
                  status = result;
                }
              });

              // 其它组件的值都直接保存
            } else {
              modulesData.push({
                id: moduleId,
                type: moduleType,
                label: moduleLabel,
                name: moduleName,
                value: moduleValue,
                require: moduleRequire
              });
            }
          }
        }
      }

      // 是否过滤表单值
      if (params === "all") {
        return { status: status, result: modulesData };
      } else {
        // 过滤结果
        let result = [];

        // 过滤空值
        for (let index = 0; index < modulesData.length; index++) {
          const element = modulesData[index];

          // 保存必填项
          if (element.require) {
            result.push(element);
            // 过滤空字符串和空对象
          } else if (
            element.value &&
            element.value !== "" &&
            element.value !== null
          ) {
            result.push(element);
            // 过滤空数组
          } else if (Array.isArray(element.value) && element.value.length > 0) {
            result.push(element);
          }
        }

        // 返回过滤结果
        return { status: status, result: result };
      }
    },

    // 处理提交
    handleSubmit() {
      // 检查结果
      let checkResult = 0;
      // 表单数量
      let formNum = 0;

      // 检查必填项前，对表单数据进行同步，将表单组件的值同步到用于验证必填项的表单组件对应的变量
      this.formDataSync("submit");

      // 计算iview表单的数量
      // 表单组件: 明细
      for (const key in this.detailedRenderData) {
        if (this.detailedRenderData.hasOwnProperty(key)) {
          formNum += 1;
        }
      }

      // 发现渲染表单
      if (this.$refs.thisForm) {
        formNum += 1;
      }

      // 渲染表单必填项检查
      this.$refs.thisForm.validate(result => {
        if (result) {
          checkResult += 1;
        }
      });

      // 表单组件: 明细，表单必填项检查
      for (const key in this.detailedRenderData) {
        if (this.detailedRenderData.hasOwnProperty(key)) {
          this.$refs[key][0].checkRequired(result => {
            if (result) {
              checkResult += 1;
            }
          });
        }
      }
      // 发现基础数据表单
      // if (this.$refs.baseDataForm) {
      //   formNum += 1;
      // }

      // 基础数据表单必填项检查
      // if (this.nowMode === this.modeOption.render || this.$refs.baseDataForm) {
      //   this.$refs.baseDataForm.validate(result => {
      //     if (result) {
      //       checkResult += 1;
      //     }
      //   });
      // }

      // 检查所有表单的必填项是否都已填写
      if (checkResult === formNum) {
        // 获取表单键值对
        let modulesData = this.getModulesData();

        // 表单组件：明细 内的表单结构数据和键值对获取结果状态
        if (modulesData.status === true) {
          // 传递表单组件数据
          this.$emit("submitEvent", modulesData.result);
          return modulesData.result;
        }
      } else {
        this.$Modal.error({
          title: "错误",
          content: "请检查必填项"
        });
      }
    },

    // 为表单必填项设置验证规则
    setRequiredRules(params) {
      if (params.require) {
        // 默认为空，未填
        this.formModel[params.name] = "";
        // 验证规则：默认数据类型
        let type = 'string';
        // 验证规则：验证触发时机
        let trigger = "blur";
        // 子验证规则
        let subRule = {max: params.max, min: params.min, message: '最大值：'+ params.max +'，最小值：'+
            params.min, trigger: 'change'};

        // 数字类型必填项：数字输入框类、金额类、计算公式
        if (params.type === this.module.inputNumber.type || params.type === this.module.money.type ||
          params.type === this.module.count.type) {
          type = "number";
          trigger = "change";

          // 计算公式没有最大值最小值限制
          // if (params.type !== this.module.count.type) {
            // console.log(1111)
            // subRule = {max: params.max, min: params.min, message: '最大值：'+ params.max +'，最小值：'+
            // params.min, trigger: 'change'};
          // }
        }

        // 时间日期
        if (params.type === this.module.dateTime.type) {
          type = "date";
          trigger = "change";
        }

        // 时间日期范围
        if (params.type === this.module.dateTimeRange.type) {
          type = "array";
          trigger = "change";
        }

        // 多选框
        if (params.type === this.module.checkbox.type) {
          type = "array";
          trigger = "change";
        }

        // 单选框
        if (params.type === this.module.radio.type) {
          trigger = "change";
        }

        // 下拉框
        if (params.type === this.module.select.type) {
          trigger = "change";
        }

        // 设置验证规则
        this.ruleValidate[params.name] = [
          {
            required: params.require,
            message: "必填",
            type: type,
            trigger: trigger,
            subRule
          }
        ];
      }
    },

    // 检查必填项
    checkRequired(fn) {
      // 同步验证数据
      this.formDataSync("checkRequired");
      this.$refs.thisForm.validate(fn);
    },

    // 设置表单键值
    setFormKeyValue() {
      // 当表单结构数据和表单组件数据都存在的场合才会执行
      if (this.formData.length > 0 && this.formModuleData.length > 0) {
        // 查找行
        for (let index1 = 0; index1 < this.formData.length; index1++) {
          // 查找列
          for (
            let index2 = 0;
            index2 < this.formData[index1].length;
            index2++
          ) {
            // 查找组件
            for (
              let index3 = 0;
              index3 < this.formData[index1][index2].length;
              index3++
            ) {
              // 目标表单组件
              let targetModule = this.formData[index1][index2][index3];

              // 设置键值
              for (let index = 0; index < this.formModuleData.length; index++) {
                const element = this.formModuleData[index];
                // 找到和键值对匹配的表单元素
                if (element.id === targetModule.id) {
                  // 处理不同表单组件的值
                  switch (element.type) {
                    // 金额组件
                    case this.module.money.type:
                      targetModule.value = element.value;
                      targetModule.cnNum = this.moneyFormat(element);
                      break;

                    // 计算公式组件
                    case this.module.count.type:
                      targetModule.value = element.value;
                      targetModule.cnNum = this.moneyFormat(element);
                      break;

                    // 时间日期组件
                    case this.module.dateTime.type:
                      targetModule.value = new Date(element.value);
                      break;

                    // 时间日期范围组件
                    case this.module.dateTimeRange.type:
                      targetModule.value = [];
                      targetModule.value[0] = new Date(element.value[0]);
                      targetModule.value[1] = new Date(element.value[1]);
                      break;

                    // 明细组件
                    case this.module.detailed.type:
                      targetModule.value = element.value;
                      break;

                      break;
                    // 默认处理
                    default:
                      targetModule.value = element.value;
                      break;
                  }
                }
              }
            }
          }
        }
      }
    },

    // 重置表单验证状态
    resetFields() {
      this.$refs.thisForm.resetFields();
    },

    // 保存可计算对象
    saveCalculator: function(params) {
      // 如果当前添加的表单组件是：数字输入框或金额
      // 则同时保存到可计算对象（calculatorList）并 触发自定义事件传递到父组件
      if (
        params.type === this.module.inputNumber.type ||
        params.type === this.module.money.type ||
        params.type === this.module.count.type
      ) {
        this.calculatorList[params.id] = params;
        // 自定义事件: 添加可计算事件
        this.$emit("findCalculator", params);
      }
    },

    // 保存发现的计算公式组件
    saveCountFormula: function(params) {
      // 保存计算公式组件
      this.countFormulaList[params.id] = params;
      // 自定义事件：发现计算公式组件
      this.$emit("findCountFormula", this.countFormulaList[params.id]);
    },

    // 处理计算公式
    handleFormulas: function(params) {
      if (Array.isArray(params) && params.length > 0) {
        // 保存筛选出来的，计算公式中包含的，明细组件内计算对象id的位置
        let formulaIndex = [];
        // 筛选出来的，计算公式中包含的，明细组件内计算对象的值
        let formulaValue = {};

        // 检查是否有效值
        let checkValue = (params)=>{
          if (params && Number(params) && !isNaN(params)) {
            return params
          } else {
            return 0
          }
        };

        // 包含明细组件内计算对象的计算公式处理结果集合
        let formulasResultArr = [];

        // 遍历计算公式
        for (let index = 0; index < params.length; index++) {
          const formulaElement = params[index];
          // 过滤出计算公式中的计算对象Id
          if (/[a-zA-Z]/g.test(formulaElement)) {
            // 如果当前是在表单构建器的构建模式下，
            // 并且根据计算公式中的计算对象id，没在计算对象列表中找到对应的计算对象时，直接退出，防止找不到对象报错
            // 一般只有在表单构建器的构建模式下，编辑表单结构时才有可能出现
            if (this.nowMode === this.modeOption.builder && !this.calculatorList[formulaElement]) {
              return false
            }

            // 根据过滤出的计算对象id，筛选出计算对象列表中对应的计算对象的值
            for (const key in this.calculatorList) {
              if (this.calculatorList.hasOwnProperty(key)) {
                // 筛选计算公式中，包含的明细组件内计算对象id
                if (this.calculatorList[key].id.match(formulaElement + "-")) {
                  // 保存筛选出来的， 计算公式中包含的，明细组件内计算对象id的位置
                  formulaIndex.push(index);
                  // 以位置下标为属性名，保存筛选出来的， 计算公式中包含的，明细组件内计算对象的值
                  if (Array.isArray(formulaValue[index])) {
                    formulaValue[index].push(checkValue(this.calculatorList[key].value));
                  } else {
                    formulaValue[index] = [];
                    formulaValue[index].push(checkValue(this.calculatorList[key].value));
                  }

                // 将非明细组件内的计算对象的值替换进计算公式中
                } else if(params[index] === this.calculatorList[key].id) {
                  params[index] = checkValue(this.calculatorList[key].value);
                }
              }
            }
          }
        }

        // 如果查到的计算公式中，包含的明细组件内计算对象的数量大于0
        // 则根据
        if (formulaIndex.length > 0) {
          // 复制当前已替换了非明细组件内计算对象id的计算公式数组
          // 用于接下来的操作，避免影响到下一次操作
          let formulasHandleResult = params.concat();
          // 计算器，用于查找同一个位置，在本轮替换中需要替换进去的值
          let counter = 0;
          // 临时处理结果，用于将乘法和除法符号替换为计算机可识别的星号和斜杠
          let tempHandleResult = "";

          // 根据计算公式中任意一个，明细组件内相同计算对象值的数量，来判断需要生成的计算公式数量
          // 因为值的数量就代表着，明细组件内有多少个相同的计算对象，而每一个相同的计算对象
          // 都应该对应一个计算公式
          for (let index = 0; index < formulaValue[formulaIndex[0]].length; index++) {
            // 通过计算对象id在计算公式中的位置，确定需要插入值的位置
            for (let index = 0; index < formulaIndex.length; index++) {
              const element = formulaIndex[index];

              // 根据计算器，确认需要插入的是第几个值
              if (counter <= formulaValue[element].length - 1) {
                formulasHandleResult[element] = formulaValue[element][counter];
              }
            }

            // 替换乘法和除法符号
            tempHandleResult = formulasHandleResult.join("").replace(/\×/g, "*");
            tempHandleResult = tempHandleResult.replace(/\÷/g, "/");

            // 保存本轮替换完成的计算公式
            formulasResultArr.push(tempHandleResult);

            // 计数器加1，修改一下轮需要替换进去的值
            counter += 1;
          }

          // 返回处理完成的计算公式
          return formulasResultArr

        // 否则，只需直接返回被替换了，非明细组件内的计算对象的值，的计算公式
        } else {
          // 复制当前已替换了非明细组件内计算对象id的计算公式数组
          // 用于接下来的操作，避免影响到下一次操作
          let formulasHandleResult = params.concat();

          // 替换乘法和除法符号
          tempHandleResult = formulasHandleResult.join("").replace(/[×]/g, "*");
          tempHandleResult = tempHandleResult.replace(/[÷]/g, "/");

          // 保存本轮替换完成的计算公式
          formulasResultArr.push(tempHandleResult);

          // 返回处理完成的计算公式
          return formulasResultArr
        }
      }
    },

    // 执行计算
    performCount: function(params = {}) {

      // 删除计算对象&计算公式对象
      if (params.action === "del" && Array.isArray(params.modulesId) && params.modulesId.length > 0) {
        for (let index = 0; index < params.modulesId.length; index++) {
          const element = params.modulesId[index];
          // 计算对象
          if (this.calculatorList[element]) {
            delete this.calculatorList[element];
          }

          // 计算公式
          if (this.countFormulaList[element]) {
            delete this.countFormulaList[element];
          }
        }
      }

      // 计算公式处理并计算
      for (const key1 in this.countFormulaList) {
        if (this.countFormulaList.hasOwnProperty(key1)) {

          // 拥有计算公式的情况下才执行计算操作
          if (this.countFormulaList[key1].countFormulas) {
            // 目标计算公式组件
            let targetFormula = this.countFormulaList[key1];
            // 修改后的计算公式结果
            let formulaResult = "";
            // 计算公式的计算结果
            let countResult = 0;

            // 处理当前计算公式组件的计算公式
            formulaResult = this.handleFormulas(targetFormula.countFormulas.split("&nbsp;"));

            // 执行计算
            try {
              // 根据计算公式计算结果
              for (let index = 0; index < formulaResult.length; index++) {
                let formula = formulaResult[index];
                countResult = Number(countResult) + Number(eval(formula));
              }
              let rounding = targetFormula.rounding === 10? 0 : targetFormula.rounding;
              targetFormula.value = Number(countResult.toFixed(rounding));
            } catch (error) {
              console.log("计算公式出错：");
              console.log(error);
            }

            // 将计算结果转换成中文大写数字金额
            this.moneyFormat(targetFormula);
          }
        }
      }

      // 计算明细内的可计算对象
      this.$emit("count", params);
    },

    // 货币格式
    moneyFormat: function(params) {
      let money = params.value;
      //汉字的数字
      let cnNums = new Array(
        "零",
        "壹",
        "贰",
        "叁",
        "肆",
        "伍",
        "陆",
        "柒",
        "捌",
        "玖"
      );
      //基本单位
      let cnIntRadice = new Array("", "拾", "佰", "仟");
      //对应整数部分扩展单位
      let cnIntUnits = new Array("", "万", "亿", "兆");
      //对应小数部分单位
      let cnDecUnits = new Array("角", "分", "毫", "厘");
      //整数金额时后面跟的字符
      let cnInteger = "";
      // let cnInteger = '整';
      //整型完以后的单位
      // let cnIntLast = params.unit;
      let cnIntLast = "元";
      //最大处理的数字
      let maxNum = this.moneyMax;
      //金额整数部分
      let integerNum;
      //金额小数部分
      let decimalNum;
      //输出的中文金额字符串
      let chineseStr = "";
      //分离金额后用的数组，预定义
      let parts;

      if (money == "") {
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

      if (money.indexOf(".") == -1) {
        integerNum = money;
        decimalNum = "";
      } else {
        parts = money.split(".");
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

          if (n == "0") {
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
      if (decimalNum != "") {
        var decLen = decimalNum.length;
        for (var i = 0; i < decLen; i++) {
          var n = decimalNum.substr(i, 1);
          if (n != "0") {
            chineseStr += cnNums[Number(n)] + cnDecUnits[i];
          }
        }
      }

      if (chineseStr == "") {
        chineseStr += cnNums[0] + cnIntLast + cnInteger;
      } else if (decimalNum == "") {
        chineseStr += cnInteger;
      }

      // 设置转换结果
      params.cnNum = chineseStr;

      if (money < 0) {
        let x = {
          value: Math.abs(money)
        };
        this.moneyFormat(x);
        params.cnNum = "负" + this.moneyFormat(x);
      }

      // 返回转换结果
      return chineseStr;
    },

    // 表单组件-明细，自增明细内原始模板组件
    plusDetailModules: function(params) {
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

    // 表单组件-明细，自减明细内原始模板组件
    minusDetailModules: function(row, rowIndex) {
      if (this.nowMode === this.modeOption.detailed) {
        let modulesId = [];
        // 保留最后一组组件,作为原始数据,不完成删完
        if (this.formData.length > 1) {
          // 删除可计算对象
          for (let index = 0; index < row[0].length; index++) {
            const module = row[0][index];
            if (this.calculatorList[module.id]) {
              delete this.calculatorList[module.id];
              modulesId.push(module.id);
            }
          }

          // 删除一组表单组件
          this.formData.splice(rowIndex, 1);

          // 自定义事件：删除明细组件内组件
          this.$emit("deleteDetailModules", {action: "del", modulesId: modulesId});
        }
      }
    },

    // 表单组件-明细 渲染数据处理
    handleDetailRenderData: function(params) {
      // 与每个表单组件-明细相对应的渲染数据
      this.detailedRenderData[params.id] = this.getFormDataTemplate();

      // 将表单组件数据填入明细组件内的表单组件
      if (
        params.value &&
        typeof params.value === "string" &&
        typeof JSON.parse(params.value) === "object"
      ) {
        // 与每个表单组件-明细相对应的渲染数据
        return (this.detailedRenderData[params.id] = JSON.parse(
          JSON.parse(params.value).renderData
        ));

      // 初次渲染表单，从表单明细组件的渲染模板（renderTemplate）属性中获取初始数据
      } else if (
        params.renderTemplate &&
        params.renderTemplate.length > 0 &&
        this.detailedRenderData[params.id][0][0].length === 0
      ) {
        for (let index = 0; index < params.renderTemplate.length; index++) {
          let temp = this.deepClone(params.renderTemplate[index]);
          // let temp = Object.assign({}, params.renderTemplate[index]);
          // 在id和name属性后新增一个唯一的随机数，以防重复
          let random1 = Math.random();
          let random2 = Math.random();
          if (random1 === random2) {
            random1 = random1 - 0.01;
          }
          temp.id = temp.id + "-" + random1.toString().split(".")[1];
          temp.name = temp.name + "-" + random2.toString().split(".")[1];
          this.detailedRenderData[params.id][0][0].push(temp);
        }

        return this.detailedRenderData[params.id];
      }
    },

    // 表单组件-明细 表单组件数据处理
    handleDetailModulesData: function(params) {
      // 将表单组件数据填入明细组件内的表单组件
      if (
        params.value &&
        typeof params.value === "string" &&
        typeof JSON.parse(params.value) === "object"
      ) {
        // 与每个表单组件-明细相对应的渲染数据
        return (this.detailedRenderData[params.id] = JSON.parse(
          JSON.parse(params.value).modulesData
        ));
      }
    }
  },

  // 生命周期钩子：实例化对象
  created() {
    // 如果当前是以表单组件-明细模式执行，
    // 则将传入的renderData赋值给formData
    // 因为以表单组件-明细模式执行时，传入的renderData不会发生变化，
    // 因此无法触发watch中的赋值操作，所有要在这里手动赋值
    // modulesData 同理
    this.formData = this.renderData;
    this.formModuleData = this.modulesData;
    this.setFormKeyValue();
  },

  // 生命周期钩子：组件渲染完毕
  mounted: function() {
    // 重置表单验证状态
    this.resetFields();
  },

  // 监测数据变化
  watch: {
    // 设置渲染数据
    renderData: function(newVal, oldVal) {
      this.formData = newVal;
      this.setFormKeyValue();
    },
    // 设置键值对
    modulesData: function(newVal, oldVal) {
      this.formModuleData = newVal;
      this.setFormKeyValue();
    }
  }
};
</script>
