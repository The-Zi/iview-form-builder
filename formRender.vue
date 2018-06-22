/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-06-08 14:41:01
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

          <Form ref="baseDataForm" :model="formBase" :rules="baseDataRule" :label-width="100"
           label-position="left">
          <!-- 标题 -->
          <Col class="form-title-wrap" :xs="24" :sm="24" :md="16" :lg="16">
            <FormItem label="标题：" prop="title">
              <Input v-model="formBase.title" placeholder="必填，请对本次申请进行说明" @keyup.native="clearSpace">
              </Input>
            </FormItem>
          </Col>

          <!-- 紧急程度 -->
          <Col class="form-status-wrap" :xs="24" :sm="24" :md="8" :lg="8">
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
          <Form ref="thisForm" :model="formModel" :rules="ruleValidate">
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
                      <!-- 设置组件的验证规则 -->
                      {{requireSet({row:rowIndex, col:colIndex, m:mIndex, module:modules})}}

                      <!-- 表单组件: 预算 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '预算'"
                       :required="modules.require" :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <RadioGroup v-model="modules.value" @on-change="formDataSync">
                          <Radio v-for="(item, index) in modules.itemList" :label="item"
                           :key="modules.name + '-' + index">{{item === 0? "预算内":"预算外"}}</Radio>
                        </RadioGroup>
                      </FormItem>

                      <!-- 表单组件：文本输入框 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '文本输入框'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <Input v-model="modules.value" @keyup.native="formDataSync"
                         @on-blur="formDataSync"></Input>
                      </FormItem>

                      <!-- 表单组件：数字输入框 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '数字输入框'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
                        :min="modules.min" @keyup.native="formDataSync" @on-blur="formDataSync">
                        </Input-number>
                      </FormItem>

                      <!-- 表单组件: 单选 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '单选'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <Radio-group v-model="modules.value" @on-change="formDataSync">
                          <Radio v-for="(item, index) in modules.itemList" :label="item"
                           :key="modules.name + '-' + index"></Radio>
                        </Radio-group>
                      </FormItem>

                      <!-- 表单组件: 多选 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '多选'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <CheckboxGroup v-model="modules.value" @on-change="formDataSync">
                          <Checkbox v-for="(item, index) in modules.itemList" :key="modules.name + index"
                          :label="item"></Checkbox>
                        </CheckboxGroup>
                      </FormItem>

                      <!-- 表单组件: 下拉选择 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '下拉选择'"
                       :label="modules.label" :prop="modules.name">

                        <!-- 组件 -->
                        <Select v-model="modules.value" @on-change="formDataSync">
                          <Option v-for="(item, index) in modules.itemList" :value="item"
                          :key="modules.name + index">
                            {{item}}
                          </Option>
                        </Select>
                      </FormItem>

                      <!-- 表单组件: 多行文本 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '多行文本'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <Input v-model="modules.value" type="textarea" :rows="4" @keyup.native="formDataSync"
                          @on-blur="formDataSync">
                        </Input>
                      </FormItem>

                      <!-- 表单组件: 时间日期范围 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '时间日期范围'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <DatePicker v-model="modules.value" type="datetimerange" format="yyyy年MM月dd日"
                         placeholder="请选择时间日期范围" @on-change="formDataSync">
                         </DatePicker>
                      </FormItem>

                      <!-- 表单组件: 时间日期 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '时间日期'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <DatePicker v-model="modules.value" type="datetime" format="yyyy年MM月dd日"
                         placeholder="请选择时间日期"  @on-change="formDataSync">
                        </DatePicker>
                      </FormItem>

                      <!-- 表单组件: 说明文本 -->
                      <FormItem class="form-grid-item" v-if="modules.type === '说明文本'"
                       :label="modules.label" :prop="modules.name">
                        <!-- 组件 -->
                        <p>{{modules.value}}</p>
                      </FormItem>
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
   min-height: $colMinH;
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
         min-height: $colMinH;
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
    name: "form-render",

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
      // 表单数据
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
    },

    // 数据
    data() {
      return {
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
      }
    },

    // 方法
    methods: {
      // 清除空格
      clearSpace(){
        this.formBase.title = this.formBase.title.replace(/(^[\s\n\t]+|[\s\n\t]|[\s\n\t]+$)/g, "");
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
        let formData = this.formData;
          for (let index1 = 0; index1 < formData.length; index1++) {
            for (let index2 = 0; index2 < formData[index1].length; index2++) {
            for (let index3 = 0; index3 < formData[index1][index2].length; index3++) {
              let target = formData[index1][index2][index3];
              let value = target.value;

              if (Array.isArray(value)) {
                // 当至少有一个数组元素的值不为空才将其同步进表单验证数据中
                // 否则值则默认为空，不让表单验证规则通过
                this.formModel[target.name] = "";
                for (let index = 0; index < value.length; index++) {
                  if (value[index]) {
                    this.formModel[target.name] = this.formModel[target.name] + "##" + value[index];
                  }
                }
              } else {
                this.formModel[target.name] = "";
                // 字符串类型的值
                if (value && typeof value === "string" && value.replace(/(^[\s\n\t]+|[\s\n\t]+$)/g, "")) {
                  this.formModel[target.name] = value.toString();
                }

                // 数字类型的值
                // if (typeof value === "number" || target.type === "数字输入框") {
                //     if (typeof value === "number") {
                //       this.formModel[target.name] = value.toString();
                //     } else {
                //       this.formModel[target.name] = "";
                //     }
                // }
                if (typeof value === "number") {
                      this.formModel[target.name] = value.toString();
                } else if (target.type === "数字输入框") {
                  this.formModel[target.name] = "";
                }

                // 日期类型的值
                // if (value && typeof value === "object" && new Date(value).getFullYear()) {
                if (value instanceof Date) {
                  this.formModel[target.name] = value.toString();
                }
              }
            }
          }
        }
      },

      // 取出表单的键值对
      getKeyValue(){
        // 遍历表单结构，把所有表单元素的 key 和 value 取出来
        for (let index1 = 0; index1 < this.formData.length; index1++) {
          for (let index2 = 0; index2 < this.formData[index1].length; index2++) {
            for (let index3 = 0; index3 < this.formData[index1][index2].length; index3++) {
              let moduleType = this.formData[index1][index2][index3].type;
              let moduleValue = this.formData[index1][index2][index3].value;
              let moduleName = this.formData[index1][index2][index3].name;

              // 将时间日期组件的value转成时间戳
              if (moduleType === "时间日期" || moduleType === "时间日期范围") {
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

              // 除时间日期组件之外的值都直接保存
              } else {
                this.formKeyValue[this.formData[index1][index2][index3].name] =
                this.formData[index1][index2][index3].value;
              }
            }
          }
        }

        return {status:true, result:this.formKeyValue}
      },

      // 处理提交
      handleSubmit() {
        // 同步验证数据
        this.formDataSync();

        // 必填项检查和传递数据
        this.$refs.baseDataForm.validate((result) => {
          if (result) {
            this.$refs.thisForm.validate((statis)=>{
              if (statis) {
                // 获取表单键值对
                let keyValue = this.getKeyValue();
                if (keyValue.status === true) {
                  // 传递表单基础数据、表单结构数据、表单键值对
                  this.$emit("sendData",this.baseData, this.formData, keyValue.result);
                }
              }
            },(err)=>{
              console.log(err)
            });
          }
        });
      },

      // 必填项设置
      requireSet(params){
        this.formModel[params.module.name] = "";

        this.ruleValidate[params.module.name] = [{required: params.module.require, message:"必填",
        type:"string",trigger: "blur"}];
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
      resetForm() {
        this.$refs.baseDataForm.resetFields();
        this.$refs.thisForm.resetFields();
      },
    },

    // 生命周期钩子：组件渲染完毕
    mounted:function () {
        // 重置表单验证状态
        this.resetForm();
    },

    // 监测数据变化
    watch: {
      // 设置基础数据
      baseData: function (newVal, oldVal) {
        this.formBase = {};
        this.formBase = newVal;
      },
      // 设置渲染数据
      renderData: function (newVal, oldVal) {
        this.formData = [];
        this.formData = newVal;
      },
      // 设置键值对
      keyValue:function(newVal, oldVal){
        this.formKeyValue = {};
        this.formKeyValue = newVal;
        this.setFormKeyValue();
      },
    },
  }
</script>
