/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-06-08 17:49:24
 */

<template>
  <!-- 表单渲染 -->
  <Row class="form-grid-wrap">
    <Form ref="tempForm" :model="tempForm" inline>
      <!-- 表单行 -->
      <Col class="form-grid-row" v-for="(row, rowIndex) in modulesData" :key="rowIndex" span="24">
        <!-- 表单行操作 -->
        <div class="form-grid-col-activer">
          <Icon @click.native="gridColMinus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="删除列"
           type="minus"></Icon>
          <Icon @click.native="gridColPlus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="增加列"
           type="plus"></Icon>
        </div>

        <!-- 表单列 -->
        <Row class="form-grid-col-wrap">
          <Col class="form-grid-col" v-for="(col, colIndex) in row"
           :span="setColSize({rowIndex:rowIndex, nowIndex:colIndex})" :key="colIndex">
            <div v-activer class="form-grid-render-zoon" @dragover="allowDrop"
             @drop="drop({rowIndex: rowIndex,colIndex:colIndex}, $event)">

              <!-- 渲染表单组件 -->
              <div class="form-modules-item" v-for="(modules, mIndex) in col" :key="'form-modules-' + mIndex">
                <!-- 表单组件: 预算 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.budget.type"
                 :required="modules.require" :label="modules.label" :prop="modules.name">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <RadioGroup v-model="modules.value">
                    <Radio v-for="(item, index) in modules.itemList" :label="item"
                     :key="modules.name + '-' + index">{{item === 0? "预算内":"预算外"}}</Radio>
                  </RadioGroup>
                </FormItem>

                <!-- 表单组件：文本输入框 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.inputText.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                      @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                      type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                      @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <Input v-model="modules.value">
                  </Input>
                </FormItem>

                <!-- 表单组件：数字输入框 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.inputNumber.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
                   :min="modules.min">
                  </Input-number>
                </FormItem>

                <!-- 表单组件: 单选 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.radio.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <RadioGroup v-model="modules.value">
                    <Radio v-for="(item, index) in modules.itemList" :label="item"
                     :key="modules.name + '-' + index"></Radio>
                  </RadioGroup>
                </FormItem>

                <!-- 表单组件: 多选 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.checkbox.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <CheckboxGroup v-model="modules.value">
                    <Checkbox v-for="(item, index) in modules.itemList" :key="modules.name + index"
                    :label="item"></Checkbox>
                  </CheckboxGroup>
                </FormItem>

                <!-- 表单组件: 下拉选择 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.select.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <Select v-model="modules.value">
                    <Option v-for="(item, index) in modules.itemList" :value="item" :key="modules.name + index">
                      {{item}}
                    </Option>
                  </Select>
                </FormItem>

                <!-- 表单组件: 多行文本 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.textarea.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <Input v-model="modules.value" type="textarea" :rows="4">
                  </Input>
                </FormItem>

                <!-- 表单组件: 时间日期范围 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.dateTimeRange.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <DatePicker
                  v-model="modules.value"
                  :type="dateFormat({mode: 'type',format: modules.format})"
                  :format="dateFormat({mode: 'format',format: modules.format})"
                  placeholder="请选择时间日期范围"></DatePicker>
                </FormItem>

                <!-- 表单组件: 时间日期 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.dateTime.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <DatePicker v-model="modules.value"
                  :type="dateFormat({mode: 'type',format: modules.format})"
                  :format="dateFormat({mode: 'format',format: modules.format})"
                  placeholder="请选择时间日期">
                  </DatePicker>
                </FormItem>

                <!-- 表单组件: 说明文本 -->
                <FormItem class="form-grid-item" v-if="modules.type === element.textRead.type"
                 :required="modules.require" :label="modules.label">
                  <!-- 组件操作 -->
                  <div class="form-grid-module-action">
                    <!-- 设置 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="设置"
                     @click.native="setModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"
                     type="ios-gear"/>
                     <!-- 删除 -->
                    <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                     @click.native="delModule({rowIndex: rowIndex, colIndex:colIndex, mIndex: mIndex})"/>
                  </div>

                  <!-- 组件 -->
                  <p>{{modules.value}}</p>
                </FormItem>
               </div>
            </div>
          </Col>
        </Row>
      </Col>
    </Form>


    <!-- 表单组件设置对话框 -->
    <Modal v-model="modulesSetingModalToogle">
      <!-- 标题 -->
      <p slot="header" class="congfig-modal-header">
        <span>{{moduleConfig.type}}组件设置</span>
      </p>

      <!-- 表单组件配置项 -->
      <Form ref="configForm" :rules="ruleValidate" :model="moduleConfig" label-position="left"
       :label-width="100">
        <!-- 设为必填项 -->
        <FormItem v-show="moduleConfig.type !== element.textRead.type" label="设为必填项：">
          <Checkbox v-model="moduleConfig.require"></Checkbox>
        </FormItem>

        <!-- 通用: 字段名 -->
        <FormItem label="字段名：" prop="name">
          <Input v-model="moduleConfig.name" @keyup.native="filterBlank" placeholder="只能输入英文"></Input>
        </FormItem>

        <!-- 通用: 标题 -->
        <FormItem label="标题：" prop="label">
          <Input v-model="moduleConfig.label" placeholder="请输入该组件的名称"></Input>
        </FormItem>

        <!-- 文本输入框配置 -->
        <template v-if="moduleConfig.type === element.inputText.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>
        </template>

        <!-- 数字输入框配置 -->
        <template v-if="moduleConfig.type === element.inputNumber.type">
          <FormItem label="默认值：">
            <Input-number style="width: 100%;" v-model="moduleConfig.value"></Input-number>
          </FormItem>

          <FormItem label="最小值：">
            <Input-number style="width: 100%;" v-model="moduleConfig.min"></Input-number>
          </FormItem>

          <FormItem label="最大值：">
            <Input-number style="width: 100%;" v-model="moduleConfig.max"></Input-number>
          </FormItem>
        </template>

        <!-- 单选配置 -->
        <template v-if="moduleConfig.type === element.radio.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 多选配置 -->
        <template v-if="moduleConfig.type === element.checkbox.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.configValue" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 下拉选择配置 -->
        <template v-if="moduleConfig.type === element.select.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 多行文本配置 -->
        <template v-if="moduleConfig.type === element.textarea.type">
          <FormItem label="默认值：">
            <Input v-model="moduleConfig.value" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 说明文本配置 -->
        <template v-if="moduleConfig.type === element.textRead.type">
          <FormItem label="默认值：">
            <Input v-model="moduleConfig.value" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 时间日期配置 -->
        <template v-if="moduleConfig.type === element.dateTime.type ||
        moduleConfig.type === element.dateTimeRange.type">
          <FormItem label="格式：">
            <RadioGroup v-model="moduleConfig.format">
               <Radio :label="1">yyyy年yy月dd日</Radio>
               <Radio :label="2">yyyy年yy月dd日 hh时mm分</Radio>
               <Radio :label="3">yyyy年yy月dd日 hh时mm分ss秒</Radio>
            </RadioGroup>
          </FormItem>
        </template>
      </Form>

      <!-- 对话框底部 -->
      <div slot="footer">
        <Button type="text" size="large" @click.native="cancelConfigModal">取消</Button>
        <Button type="primary" size="large" @click.native="saveModuleConfig">确定</Button>
      </div>
    </Modal>
  </Row>
</template>

<style lang="scss">
// =============== 导入样式文件 ===============
@import "../styles/baseStyles";
@import "../styles/commonStyles";


  // 表单项渲染区域
 .form-grid-wrap {
   min-height: $colMinH;
   // 表单栅格行
  .form-grid-row {
    border-bottom: 1px solid #ccc;
    &:last-child {
      border-bottom: none;
    }
    // 表单行操作
    .form-grid-col-activer {
      position: absolute;
      top: 0;
      right: -25px;
      width: 20px;
      height: 60px;
      padding: $padding;
      .activer {
        display: block;
        width: 20px;
        height: 20px;
        margin-bottom: 5px;
        cursor: pointer;
      }
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
            // 表单组件设置操作
            .form-grid-module-action {
              position: absolute;
              right: 0;
              z-index: 999;
              // position: relative;
              // margin-top: -30px;
              // float: right;
              // 操作
              > .action {
                display: inline-block;
                width: 20px;
                height: 20px;
                margin-left: 5px;
                cursor: pointer;
              }
            }
           }
         }
         .form-grid-col-message {
           position: absolute;
           top: 50%;
           left: 50%;
           width: 100px;
           height: 20px;
           margin-left: -50px;
           margin-top: -10px;
           line-height: 20px;
           text-align: center;
         }
       }
     }
    }
  }

  // 对话框标题
 .congfig-modal-header {
  font-weight: normal;
  font-size: 16px;
 }
</style>

<script>
// =============== 导入模块 ===============
// 配置文件
import config from '../config';
// 上传组件
import formUpload from '../formUpload'
  export default {
    // 组件名
    name: "iview-form-builder-grid",

    // 组件
    components:{
      formUpload
    },

    // 自定义属性
    props: {
      // 行数
      rowNumber: {
        default: 1
      },
    },

    // 自定义指令
    directives: {
      // 活动中
      activer: {
        inserted: (el) => {
          // 添加提示色
          function activerOn() {
            el.style.backgroundColor = "#e6e6e6";
          }

          // 清除提示色
          function activerOff() {
            el.style.backgroundColor = "#fff";
          }

          el.ondragenter = activerOn;
          el.ondragleave = activerOff;
          el.ondragover = activerOn;
          el.ondrop = activerOff;
        }
      }
    },

    // 数据
    data() {
      return {
        // 表单组件数据
        modulesData: [
          // 默认行
          [
            // 默认列
            []
          ]
        ],
        // 临时表单数据，占位用，防止报错
        tempForm: {},
        element: config.formElement,
        // 表单组件设置对话框
        modulesSetingModalToogle: false,
        // 表单组件配置
        moduleConfig: {},
        // 暂存当前配置中的组件
        targetModule: "",
        // 组件配置验证配置
        ruleValidate: {
          name: [
              {required: true, message: "必填，只能输入英文字母和数字", trigger: "blur",pattern: /[0-9A-Za-z]/},
          ]
        },
      }
    },

    // 方法
    methods: {
      // 阻止被放置数据的元素的默认行为
      allowDrop(ev){
        ev.preventDefault();
      },

      // 拖放添加表单组件
      drop:function(params, $event){
        // 获取拖动传递过来的值
        let module = $event.dataTransfer.getData("Text");
        // 生成时间戳，用于区分不同的字段名
        let tempDate = new Date();
        // 临时表单组件项
        let tempItem = {};

        tempItem.type = module;
        tempItem.value = "";
        tempItem.defaultValue = "";
        tempItem.require = false;
        tempItem.name = "name" + tempDate.getTime();

        // 配置并添加表单组件
        switch (module) {
          case this.element.budget.type:
            tempItem.label= this.element.budget.title + "：";
            tempItem.value = null;
            tempItem.name = "is_excess_budget";
            tempItem.require = true;
            tempItem.itemList = [0, 1];
            break;

          case this.element.inputText.type:
            tempItem.label = this.element.inputText.title + "：";
            // tempItem.placeholder = "请输入文本";
            break;

          case this.element.inputNumber.type:
            tempItem.label = this.element.inputNumber.title + "：";
            // tempItem.placeholder = "请输入数字";
            tempItem.value = null;
            tempItem.max = 20000;
            tempItem.min = 0;
            break;

          case this.element.radio.type:
            tempItem.label = this.element.radio.title + "：";
            tempItem.value = "";
            tempItem.itemList = ["选项1", "选项2", "选项3"];
            break;

          case this.element.checkbox.type:
            tempItem.label = this.element.checkbox.title + "：";
            tempItem.value = [];
            tempItem.itemList = ["选项1", "选项2", "选项3","选项4","选项5","选项6","选项7","选项8"];
            break;

          case this.element.select.type:
            tempItem.label = this.element.select.title + "：";
            tempItem.value = "";
            tempItem.itemList = ["选项1", "选项2", "选项3"];
            break;

          case this.element.textarea.type:
            tempItem.label = this.element.textarea.title + "：";
          break;

          case this.element.dateTime.type:
            tempItem.label = this.element.dateTime.title + "：";
            tempItem.format = 1;
          break;

          case this.element.dateTimeRange.type:
            tempItem.label = this.element.dateTimeRange.title + "：";
            tempItem.format = 1;
          break;

          case this.element.textRead.type:
            tempItem.label = this.element.textRead.title + "：";
            tempItem.value = "一些说明";
          break;
        }

        // 添加新表单组件
        this.modulesData[params.rowIndex][params.colIndex].push(tempItem);
      },

      // 日期格式设置
      dateFormat(params){
          // 时间日期格式
          switch (params.format) {
            case 1:
                if (params.mode === "format") {
                  return "yyyy年MM月dd日"
                } else if (params.mode === "type") {
                  return "date"
                }
              break;

            case 2:
                if (params.mode === "format") {
                  return "yyyy年MM月dd日 HH:mm"
                } else if (params.mode === "type") {
                  return "datetime"
                }
            break;

            case 3:
                if (params.mode === "format") {
                  return ""
                } else if (params.mode === "type") {
                  return "datetime"
                }
            break;
          }
      },

      // 组件设置
      setModule: function (params) {
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;
        let module = this.modulesData[row][col][m];

        // 显示组件配置
        // 复制组件属性新的变量，防止修改的时候互相干扰
        for (const key in module) {
          if (module.hasOwnProperty(key)) {
            this.moduleConfig[key] = module[key];
          }
        }

        // 单选框组件、多选框组件、下拉选择组件配置
        if(module.type === this.element.radio.type || module.type === this.element.checkbox.type ||
        module.type === this.element.select.type) {
          this.moduleConfig.editList = module.itemList.join("##");
        }

        // 数字输入框组件配置
        if (module.type === this.element.inputNumber.type) {
          this.moduleConfig.min = module.min;
          this.moduleConfig.max = module.max;
        }

        // 修改配置的时候默认值需要的是字符串或者数组，但是checkbox组件需要的是数组
        // 所以在里专门添加一个属性来存放，转为字符串的checkbox值
        if(module.type === this.element.checkbox.type) {
          this.moduleConfig.configValue = module.value.join("##");
        }

        // 暂存当前修改配置中的组件
        this.targetModule = params;

        // 显示组件配置对话框
        this.modulesSetingModalToogle = true;
      },

      // 保存组件配置
      saveModuleConfig() {
        let params = this.targetModule;
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;
        let module = this.modulesData[row][col][m];

        // 组件特有配置处理
        // 单选框组件、多选框组件、下拉选择组件配置
        if(module.type === this.element.radio.type || module.type === this.element.checkbox.type ||
        module.type === this.element.select.type) {
          this.moduleConfig.itemList = this.moduleConfig.editList.split('##');
        }

        // 多选框组件配置
        if( module.type === this.element.checkbox.type) {
          this.moduleConfig.value = this.moduleConfig.configValue.split('##');
        }

        // 配置验证正确，关闭对话框
        this.$refs.configForm.validate((result)=>{
          if (result) {
            // 更新数据
            for (const key in this.moduleConfig) {
              if (this.moduleConfig.hasOwnProperty(key)) {
                if(key === "require" || this.modulesData[row][col][m][key] !== undefined){
                  this.modulesData[row][col][m][key] = this.moduleConfig[key];
                }
              }
            }
            // 退出组件配置对话框
            this.cancelConfigModal();
          }
        })
      },

      // 退出组件配置对话框
      cancelConfigModal() {
        // 清空验证信息
        this.$refs.configForm.resetFields();
        // 关闭对话框
        this.modulesSetingModalToogle = !this.modulesSetingModalToogle;
      },

      // 删除组件
      delModule: function (params) {
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;

        // 删除指定组件
        this.modulesData[row][col].splice(m,1);
      },

      // 增加表单行列数
      gridColPlus(colIndex) {
        if (this.modulesData[colIndex].length < 8) {
          this.modulesData[colIndex].push([])
        }
      },

      // 删除表单行
      gridColMinus(colIndex) {
        let col = this.modulesData[colIndex];
        let collength = this.modulesData[colIndex].length;

        // 保留一行不删完
        if (collength > 1 ) {
          this.modulesData[colIndex].splice(-1,1);
        }
      },

      // 清空当前构建的表单
      clearBuilder(){
        this.modulesData = [[[]]];
        this.moduleConfig = {};
      },

      // 传递构建的表单数据
      saveBuilder: function () {
        this.$emit("save", {data:this.modulesData});
      },

      // 设置列大小
      setColSize(params) {
        // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
        // 前提是必须在html文件的<head>内添加
        // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
        let deviceWidth =  window.screen.width;
        if (deviceWidth && deviceWidth < 992) {
          return 24
        }

        let col = this.modulesData[params.rowIndex];
        let collength = this.modulesData[params.rowIndex].length;
        let size = 24 / collength;

        if (size.toString().split(".").length > 1) {
          if (params.nowIndex === collength - 1) {
            return 24 / (collength + 1) * 2
          } else {
            return 24 / (collength + 1)
          }
        } else {
            return 24 / collength
        }
      },

      // 过滤空格
      filterBlank: function () {
        this.moduleConfig = Object.assign({},this.moduleConfig,
         {name:this.moduleConfig.name.replace(/([\u4e00-\u9fa5]{1,}|[\s+]{1,})/,"")});
      }
    },

    // 生命周期钩子：组件加载完毕
    mounted: function () {
        // 防止浏览器，拖动元素，松开鼠标后，会在新窗口打开被拖动的元素或传递的数据，的默认行为
      document.body.ondrop = function (param) {
        param.preventDefault();
        param.stopPropagation();
      };
    },

    // 监视数据变动
    watch: {
      // 渲染表单行
      rowNumber: function (newValue, oldValue) {
        if(newValue > oldValue){
          let number = newValue - oldValue;
          for (let index = 0; index < number; index++) {
            this.modulesData.push([[]]);
          }
        } else if (newValue < oldValue) {
          let number = oldValue - newValue;
          this.modulesData.splice(-1, number);
        }
      }
    },
  }
</script>
