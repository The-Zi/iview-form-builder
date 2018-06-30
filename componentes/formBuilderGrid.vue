/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-06-28 18:06:01
 */

<template>
  <!-- 表单构建器：栅格系统 -->
  <Row class="form-grid-wrap">
    <Form ref="tempForm" :model="tempForm" inline>
      <!-- 表单行 -->
      <Col class="form-grid-row" v-for="(row, rowIndex) in elementsData" :key="rowIndex" span="24">
        <!-- 表单列增减 -->
        <div class="form-grid-col-activer" v-if="mode !== modeOption.element.type">
          <!-- 刪除列 -->
          <Icon @click.native="gridColMinus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="删除列"
          type="minus"></Icon>

          <!-- 增加列 -->
          <Icon @click.native="gridColPlus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="增加列"
          type="plus"></Icon>
        </div>
        <!-- 表单列 -->
        <Row class="form-grid-col-wrap">
          <Col class="form-grid-col" v-for="(col, colIndex) in row"
           :span="setColSize({rowIndex:rowIndex, nowIndex:colIndex})" :key="colIndex">
            <div v-activer class="form-grid-render-zoon" @dragover="allowDrop"
             @drop="drop({rowIndex: rowIndex,colIndex:colIndex}, $event)">
                <!-- =============== 提示信息 =============== -->
                <p class="form-grid-col-tips" v-if="!col || col.length <= 0">
                  可拖入多个组件<span v-if="mode === modeOption.element.type">（不包含明细组件）</span>
                </p>

                <!-- =============== 表单元素渲染 =============== -->
                <form-builder-elements :elementsData="{rowIndex: rowIndex,colIndex:colIndex, col: col}"
                @delete="delModule" @setting="setModule" @moneyFormat="moneyFormat"></form-builder-elements>

                <!-- =============== 表单元素：明细 =============== -->
                <div v-if="mode !== modeOption.element.type" >
                  <div v-for="(modules, mIndex) in col" :key="mIndex">
                    <FormItem class="form-elements-wrap" v-if="modules.type === element.detailed.type"
                    :required="modules.require" :label="modules.label">
                        <!-- 操作元素 -->
                        <div class="form-elements-action">
                          <!-- 设置 -->
                          <Icon class="action" size="16" color="#a7a7a7" title="设置"
                          @click.native="setModule({rowIndex: rowIndex, colIndex: colIndex, mIndex: mIndex,
                          element: modules})"
                          type="ios-gear"/>
                          <!-- 删除 -->
                          <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                          @click.native="delModule({rowIndex: rowIndex, colIndex: colIndex, mIndex: mIndex,
                          element: modules})"/>
                        </div>

                        <!-- 设置 表单元素：明细 的渲染数据 -->
                        <!-- {{setDetailRenderData(modules)}} -->

                        <!-- 通过递归当前组件，实现表单元素：明细 -->
                        <!-- <iview-form-builder-grid :data="modules.childElements" -->
                        <!-- <iview-form-builder-grid :data="renderData" -->
                        <iview-form-builder-grid :data="detailedData[modules.id]"
                        :mode="modeOption.element.type"
                        @addElements="addDetailElement(modules, $event)"
                        @setElements="setDetailElement(modules, $event)"
                        @delElements="delDetailElement(modules, $event)">
                        </iview-form-builder-grid>

                        <!-- 表单元素：明细，操作 -->
                        <div class="form-elements-detailed-action">
                            <ButtonGroup>
                                <!-- 移除增加的元素 -->
                                <Button type="ghost" icon="minus" @click.native="minusDetailElements(modules)">
                                  {{modules.actionMinus}}
                                </Button>
                                <!-- 循环复制增加元素 -->
                                <Button type="ghost" icon="plus" @click.native="plusDetailElements(modules)">
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

    <!-- =============== 栅格系统：增减行数 =============== -->
    <Col class="form-grid-row-action" span="24" v-if="mode !== modeOption.element.type">
      <ButtonGroup shape="circle">
          <Button type="ghost" @click.native="rowMinus" title="删除行">
            &nbsp;
            &nbsp;
            &nbsp;
            <Icon type="minus"></Icon>
            &nbsp;
            &nbsp;
            &nbsp;
          </Button>
          <Button type="ghost" @click.native="rowPlus" title="增加行">
            &nbsp;
            &nbsp;
            &nbsp;
            <Icon type="plus"></Icon>
            &nbsp;
            &nbsp;
            &nbsp;
          </Button>
      </ButtonGroup>
    </Col>

    <!-- =============== 对话框：表单元素设置 =============== -->
    <Modal v-model="modulesSetingModalToogle" :closable="false" :mask-closable="false">
      <!-- 标题 -->
      <p slot="header" class="congfig-modal-header">
        <span>{{moduleConfig.title}}设置</span>
      </p>

      <!-- 表单组件配置项 & 计算公式配置项-->
      <!-- 在同一对话框切换显示表单元素配置项和计算公式配置项 -->
      <Form v-show="!formulasSettingToogle" ref="configForm" :rules="ruleValidate" :model="moduleConfig" label-position="left"
       :label-width="100">

        <!-- 设为必填项 -->
        <!-- 表单元素：说明文本 和 明细，不需要必填项设置 -->
        <FormItem v-if="moduleConfig.type !== element.textRead.type &&
        moduleConfig.type !== element.detailed.type" label="设为必填项：">
          <Checkbox v-model="moduleConfig.require"></Checkbox>
        </FormItem>

        <!-- 字段名 -->
        <!-- 表单元素： 明细，不需要字段名设置 -->
        <FormItem label="字段名：" v-if="moduleConfig.type !== element.detailed.type" prop="name">
          <Input v-model="moduleConfig.name" @keyup.native="filterBlank" placeholder="只能输入英文"></Input>
        </FormItem>

        <!-- 标题 -->
        <FormItem label="标题：" prop="label">
          <Input v-model="moduleConfig.label" :maxlength="20" placeholder="请输入该组件的名称"></Input>
        </FormItem>

        <!-- 文本输入框配置 -->
        <template v-if="moduleConfig.type === element.inputText.type">
          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>
        </template>

        <!-- 数字输入框配置 -->
        <template v-if="moduleConfig.type === element.inputNumber.type">
          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="默认值：">
            <Input-number style="width: 100%;" :min="moduleConfig.min" :max="moduleConfig.max"
            v-model="moduleConfig.value"></Input-number>
          </FormItem>

          <FormItem label="最小值：">
            <Input-number style="width: 100%;" :max="moduleConfig.max" v-model="moduleConfig.min"></Input-number>
          </FormItem>

          <FormItem label="最大值：">
            <Input-number style="width: 100%;" :min="moduleConfig.min" v-model="moduleConfig.max"></Input-number>
          </FormItem>
        </template>

        <!-- 计算公式 配置 -->
        <template v-if="moduleConfig.type === element.count.type">
          <FormItem label="计算公式：">
            <Input v-model="moduleConfig.formulas" placeholder="结果 = " :readonly="true"
            @click.native="setFormulasModal">
            </Input>
          </FormItem>

          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="显示大写：">
            <Checkbox v-model="moduleConfig.showUpper"></Checkbox>
          </FormItem>

          <!-- <FormItem label="大写结果提示：">
            <Input style="width: 100%;" v-model="moduleConfig.resultTips" placeholder="最大长度10"
            :maxlength="10"></Input>
          </FormItem>

          <FormItem label="单位提示：">
            <Input style="width: 100%;" v-model="moduleConfig.unit" placeholder="最大长度20"
            :maxlength="20"></Input> -->
          </FormItem>
        </template>

        <!-- 金额 -->
        <template v-if="moduleConfig.type === element.money.type">
          <FormItem label="默认值：">
            <Input-number style="width: 100%;" :min="moduleConfig.min" :max="moduleConfig.max"
            v-model="moduleConfig.value"></Input-number>
          </FormItem>

          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="显示大写：">
            <Checkbox v-model="moduleConfig.showUpper"></Checkbox>
          </FormItem>

          <FormItem label="最小值：">
            <Input-number style="width: 100%;" :min="0" :max="this.moneyMax" v-model="moduleConfig.min"></Input-number>
          </FormItem>

          <FormItem label="最大值：">
            <Input-number style="width: 100%;" :min="0" :max="this.moneyMax" v-model="moduleConfig.max"></Input-number>
          </FormItem>

          <!-- <FormItem label="大写结果提示：">
            <Input style="width: 100%;" v-model="moduleConfig.resultTips" placeholder="最大长度10"
            :maxlength="10"></Input>
          </FormItem> -->

          <!-- <FormItem label="单位提示：">
            <Input style="width: 100%;" v-model="moduleConfig.unit" placeholder="最大长度10"
            :maxlength="10"></Input> -->
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

          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>
        </template>

        <!-- 说明文本配置 -->
        <template v-if="moduleConfig.type === element.textRead.type">
          <FormItem label="默认值：">
            <Input v-model="moduleConfig.value" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 时间日期 与 时间日期范围 配置 -->
        <template v-if="moduleConfig.type === element.dateTime.type ||
        moduleConfig.type === element.dateTimeRange.type">
          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="时间日期格式：">
            <Select v-model="moduleConfig.format">
              <Option :value="1">年-月-日</Option>
              <Option :value="2">年-月-日 时：分</Option>
            </Select>
          </FormItem>
        </template>

        <!-- 明细 配置 -->
        <template v-if="moduleConfig.type === element.detailed.type">
          <FormItem label="动作名称（增加）：" prop="plus">
            <Input v-model="moduleConfig.actionPlus" placeholder="增加明细的动作名称"></Input>
          </FormItem>
          <FormItem label="动作名称（减少）：" prop="minus">
            <Input v-model="moduleConfig.actionMinus" placeholder=""></Input>
          </FormItem>
        </template>
      </Form>

      <!-- 对话框底部 -->
      <div slot="footer" v-show="!formulasSettingToogle">
        <Button type="ghost" size="large" @click.native="cancelConfigModal">关闭</Button>
        <Button type="primary" size="large" @click.native="saveModuleConfig">确定</Button>
      </div>

      <!-- 计算公式 配置 -->
      <template v-show="formulasSettingToogle">
        <div v-show="formulasSettingToogle">
          <h3>计算公式 配置</h3>
        </div>

        <div slot="footer" v-show="formulasSettingToogle">
          <Button type="ghost" size="large" @click.native="closeFormulasSetting">取消</Button>
          <Button type="primary" size="large" @click.native="saveFormulasSetting">保存计算公式</Button>
        </div>
      </template>
    </Modal>
  </Row>
</template>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/baseStyles";
@import "../styles/commonStyles";


  // 表单项渲染区域
 .form-grid-wrap {
    min-height: $colMinH;
    // 表单单元格行数增加
    .form-grid-row-action {
      text-align: center;
      padding-top: $padding;
    }
    // 表单栅格行
    .form-grid-row {
      border: 1px solid #ccc;
      border-bottom: 0;
      &:last-child {
        border: 1px solid #ccc;
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
          // 帮助信息
          .form-grid-col-tips {
            height: 100%;
            padding: 0 $padding;
            color: #b7b7b7;
            text-align: center;
            font-size: 14px;
            line-height: $colMinH;
            border-radius: 4px;
            border: 2px dotted #ccc;
          }
          // 表单渲染区
          .form-grid-render-zoon {
            height: 100%;
            .form-grid-render-activer {
              height: 100%;
              border: 1px dashed #adadad;
            }
            // 表单元素项
            .form-elements-wrap {
              width: 100%;
              // 表单元素操作
              .form-elements-action {
                  position: absolute;
                  right: 0;
                  z-index: 999;
                  // 操作
                  > .action {
                    display: inline-block;
                    width: 20px;
                    height: 20px;
                    margin-left: 5px;
                    cursor: pointer;
                  }
              }
              // 操作
              .form-elements-detailed-action {
                padding-top: $padding;
                text-align: center;
              }
            }
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
// 表单元素
import formBuilderElements from './formBuilderElements';

  export default {
    // 组件名
    name: "iview-form-builder-grid",

    // 组件
    components:{
      formBuilderElements
    },

    // 自定义属性
    props: {
      // 当前组件的模式
      mode: {
        default: null
      },
      // 渲染用表单数据
      data: {
        default: Array
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
        // 表单数据模板，用于渲染表单的数据结构应该与其一致
        formDataTemplate: [[[]]],

        // 表单组件数据
        elementsData: [
          // 默认行
          [
            // 默认列
            []
          ]
        ],
        // 表单行数
        rowNumber: 1,
        // 临时表单数据，占位用，防止报错
        tempForm: {},
        element: config.formElement,
        // 表单组件设置对话框
        modulesSetingModalToogle: false,
        // 计算公式 配置
        formulasSettingToogle: false,
        // 表单组件配置
        moduleConfig: {},
        // 暂存当前配置中的组件
        targetModule: "",
        // 表单元素配置验证规则
        ruleValidate: {
          name: [
              {required: true, message: "必填，只能输入英文字母和数字，并只能以英文字母开头", trigger: "blur",
              pattern: /[0-9A-Za-z]/},
          ]
        },
        // 金额最大值
        moneyMax: 999999999999999.9999,
        // 表单元素数量计数器
        elementsLength: {},
        // 表单元素数量唯一下标计数器
        elementsIndex: {},
        // 组件模式选项
        modeOption: {
          element: {
            type: "detailed"
          },
          renderMode: {
            rendering: "render"
          }
        },
        // 表单元素：明细，表单数据
        detailedData: {},
        // 表单组件数据
        renderData: [
          // 默认行
          [
            // 默认列
            []
          ]
        ],
      }
    },

    // 计算属性
    // computed: {
    //   // 不能直接操作props属性
    //  renderData: {
    //    get: function () {
    //      return [[this.data]]
    //    },
    //    set: function (newVal) {
    //      this.data = [[newVal]];
    //    }
    //  }
    // },

    // 方法
    methods: {
      // 阻止被放置数据的元素的默认行为
      allowDrop: function (ev) {
        ev.preventDefault();
      },

      // 拖放添加表单组件
      drop: function (params, $event) {
        // 获取拖动传递过来的值
        let element = $event.dataTransfer.getData("Text");
        // 生成时间戳，用于区分不同的字段名
        let tempDate = new Date();
        // 临时表单元素项
        let tempItem = {};

        // 表单元素类型
        tempItem.type = element;
        // 唯一识别ID
        tempItem.id = element + "-" + tempDate.getTime();
        // 表单元素值
        tempItem.value = "";
        // 该表单元素是否为必填项
        tempItem.require = false;
        // 默认表单key
        tempItem.name = element + tempDate.getTime();

        // 配置并添加表单组件
        switch (element) {
          // 预算
          case this.element.budget.type:
            tempItem.title = this.element.budget.title;
            tempItem.label= tempItem.title + "：";
            tempItem.value = "";
            tempItem.name = "is_excess_budget";
            tempItem.require = true;
            tempItem.itemList = [0, 1];
            break;

          // 文本输入框
          case this.element.inputText.type:
            tempItem.title = this.element.inputText.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            // 表单元素提示信息/占位符
            tempItem.tips = "";
            break;

          // 数字输入框
          case this.element.inputNumber.type:
            tempItem.title = this.element.inputNumber.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            // 表单元素提示信息/占位符
            tempItem.tips = "";
            tempItem.value = null;
            tempItem.max = 20000;
            tempItem.min = 0;
            break;

          // 计算公式
          case this.element.count.type:
            tempItem.title = this.element.count.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            // 表单元素提示信息/占位符
            tempItem.tips = "";
            tempItem.value = null;
            tempItem.formulas = "";
            tempItem.resultTips = "大写：";
            tempItem.unit = "";
            tempItem.cnNum = "";
            tempItem.showUpper = true;
          break;

          // 金额
          case this.element.money.type:
            tempItem.title = this.element.money.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            // 表单元素提示信息/占位符
            tempItem.tips = "";
            tempItem.value = null;
            tempItem.max = 100000;
            tempItem.min = 0;
            tempItem.resultTips = "大写：";
            tempItem.unit = "";
            tempItem.cnNum = "";
            tempItem.showUpper = true;
          break;

          // 单选框
          case this.element.radio.type:
            tempItem.title = this.element.radio.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.value = "";
            tempItem.itemList = ["选项1", "选项2", "选项3"];
            break;

          // 复选框
          case this.element.checkbox.type:
            tempItem.title = this.element.checkbox.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.value = [];
            tempItem.itemList = ["选项1", "选项2", "选项3","选项4","选项5","选项6","选项7","选项8"];
            break;

          // 下拉框
          case this.element.select.type:
            tempItem.title = this.element.select.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.value = "";
            tempItem.itemList = ["选项1", "选项2", "选项3"];
            break;

          // 多行文本
          case this.element.textarea.type:
            tempItem.title = this.element.textarea.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.tips = "";
          break;

          // 说明信息
          case this.element.textRead.type:
            tempItem.title = this.element.textRead.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.value = "说明信息";
          break;

          // 时间日期
          case this.element.dateTime.type:
            tempItem.title = this.element.dateTime.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.tips = "";
            tempItem.format = 1;
          break;

          // 时间日期范围
          case this.element.dateTimeRange.type:
            tempItem.title = this.element.dateTimeRange.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.tips = "";
            tempItem.format = 1;
          break;

          // 明细
          case this.element.detailed.type:
            tempItem.title = this.element.detailed.title;

            // 在表单元素label后面加上数量
            // 要先设置title属性
            tempItem.label = this.countElementsNum(tempItem);

            tempItem.childElements = [];
            tempItem.actionPlus = "增加";
            tempItem.actionMinus = "减少";
            // 添加表单元素：明细 渲染用数据
            this.detailedData[tempItem.id] = this.formDataTemplate.slice(0);
          break;
        }

        // 不允许在表单元素：明细，内添加表单元素：明细
        if (this.mode === this.modeOption.element.type) {
          if (tempItem.type !== this.element.detailed.type) {
            // 添加新表单组件
            if (this.mode === this.modeOption.element.type) {
              // 触发添加元素事件，以对象形式传递新添加的表单元素
              this.$emit("addElements", tempItem);
            }
            this.elementsData[params.rowIndex][params.colIndex].push(tempItem);
          }
        } else {
          // 添加新表单组件
          this.elementsData[params.rowIndex][params.colIndex].push(tempItem);
        }

        // 阻止默认行为和事件冒泡，防止触发表单构建器的拖放事件
        $event.preventDefault();
        $event.stopPropagation();
      },

      // 计算各表单元素数量
      countElementsNum: function (params) {
        if (this.elementsIndex[params.type] >= 0) {
          this.elementsIndex[params.type] =  this.elementsIndex[params.type] + 1;
          this.elementsLength[params.type] = this.elementsLength[params.type] + 1;
        } else {
          this.elementsIndex[params.type] = 0
          this.elementsLength[params.type] = 0;
        }

        return params.title + " " +
        (this.elementsIndex[params.type] === 0 ? "" : this.elementsIndex[params.type]) + "：";
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
        // let cnInteger = '';
        let cnInteger = '整';
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
      },

      // 显示计算公式配置
      setFormulasModal: function () {
        this.formulasSettingToogle = true;
      },

      // 关闭计算公式配置
      closeFormulasSetting: function () {
        this.formulasSettingToogle = false;
      },

      // 保存计算公式配置
      saveFormulasSetting: function () {
        console.log(22)
      },

      // 组件设置
      setModule: function (params) {
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;
        let module = this.elementsData[row][col][m];

        // 显示组件配置
        // 复制组件属性新的变量，防止修改的时候互相干扰
        // for (const key in module) {
          // if (module.hasOwnProperty(key)) {
            // this.moduleConfig[key] = module[key];
          // }
        // }
        this.moduleConfig = Object.assign({}, this.moduleConfig, module);

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
      saveModuleConfig: function () {
        let params = this.targetModule;
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;
        let module = this.elementsData[row][col][m];

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
                if(key === "require" || key === "tips" || this.elementsData[row][col][m][key] !== undefined){
                  this.elementsData[row][col][m][key] = this.moduleConfig[key];
                }
              }
            }

            // 触发表单元素设置事件，以对象形式传递新添加的表单元素
            this.$emit("setElements", this.elementsData[row][col][m]);

            // 退出组件配置对话框{
              this.cancelConfigModal();
          }
        })
      },

      // 退出组件配置对话框
      cancelConfigModal: function () {
        // 清空验证信息
        this.$refs.configForm.resetFields();

        // 关闭计算公式配置项
        this.formulasSettingToogle = false;

        // 关闭对话框
        this.modulesSetingModalToogle = false;
      },

      // 删除组件
      delModule: function (params) {
        let row = params.rowIndex;
        let col = params.colIndex;
        let m = params.mIndex;

        // 触发删除表单元素事件，以对象形式传递新添加的表单元素
        if (this.mode === this.modeOption.element.type) {
          this.$emit("delElements", this.elementsData[row][col][m]);
        }

        // 删除指定组件
        this.elementsData[row][col].splice(m,1);
        // 减少该组件的数量计数器
        this.elementsLength[params.element.type] = this.elementsLength[params.element.type] - 1;

        // 重置表单元素唯一下标计数器
        if (this.elementsLength[params.element.type] === 0) {
          this.elementsIndex[params.element.type] = undefined;
        }
      },

      // 增加表单行
      rowPlus: function () {
        this.rowNumber = this.rowNumber + 1;
      },

      // 减少表单行
      rowMinus: function () {
        if (this.rowNumber > 1) {
          this.rowNumber = this.rowNumber - 1;
        }
      },

      // 增加表单行列数
      gridColPlus: function (colIndex) {
        if (this.elementsData[colIndex].length < 8) {
          this.elementsData[colIndex].push([])
        }
      },

      // 删除表单行列数
      gridColMinus: function (colIndex) {
        let col = this.elementsData[colIndex];
        let collength = this.elementsData[colIndex].length;

        // 保留一行不删完
        if (collength > 1 ) {
          this.elementsData[colIndex].splice(-1,1);
        }
      },

      // 清空当前构建的表单
      clearBuilder: function () {
        this.elementsData = this.formDataTemplate.slice(0);
        this.moduleConfig = {};
        this.elementsLength = {};
      },

      // 传递构建的表单数据
      saveBuilder: function () {
        this.$emit("save", {data:this.elementsData});
      },

      // 设置列大小
      setColSize: function (params) {
        // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
        // 前提是必须在html文件的<head>内添加
        // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
        let deviceWidth =  window.screen.width;
        if (deviceWidth && deviceWidth < 992) {
          return 24
        }

        let col = this.elementsData[params.rowIndex];
        let collength = this.elementsData[params.rowIndex].length;
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
        this.moduleConfig = Object.assign({}, this.moduleConfig,
        //  {name:this.moduleConfig.name.replace(/([\u4e00-\u9fa5]{1,}|[\s+]{1,})/,"")});
         {name:this.moduleConfig.name.replace(/(^[\d]{1,}|[^\da-zA-Z]{1,})/,"")});
      },

      // 设置表单元素：明细 对应的渲染数据
      setDetailRenderData: function (params) {
        this.detailedData[params.id] = this.formDataTemplate.slice(0);
      },

      // 保存 表单元素：明细 内的表单元素
      addDetailElement: function (params, $event) {
        let newDetailedData = Object.assign({},$event)
        let newChildElements = Object.assign({},$event)

        // 将新表单元素添加到 表单元素：明细 的子元素属性中
        params.childElements.push($event);

        // 将新表单元素添加到 表单元素：明细 的渲染数据中
        this.detailedData[params.id][0][0].push($event);
      },

      // 修改 表单元素：明细 内的表单元素
      setDetailElement: function (params, $event) {
        for (let index = 0; index < params.childElements.length; index++) {
          if (params.childElements[index].id === $event.id) {
            params.childElements[index] = Object.assign({}, params.childElements[index], $event);
          }
        }
      },

      // 删除 表单元素：明细 内的表单元素
      delDetailElement: function (params, $event) {
        // 删除 渲染用原始数据
        for (let index = 0; index < params.childElements.length; index++) {
          if (params.childElements[index].id === $event.id) {
            params.childElements.splice(index, 1);
          }
        }

        // 删除增删用数据
        for (let index = 0; index < this.detailedData[params.id][0][0].length; index++) {
          if (this.detailedData[params.id][0][0][index].id === $event.id) {
            this.detailedData[params.id][0][0].splice(index, 1);
          }
        }
      },

      // 增加 表单元素：明细 内的表单元素
      plusDetailElements: function (params) {
        if (this.mode !== this.modeOption.renderMode.rendering) {
          // 旧的渲染数据
          let oldData;
          // 处理结果
          let result = this.formDataTemplate.slice(0);

          // 如果存在旧的渲染数据，就先将旧的数据独立出来保存
          if (this.detailedData[params.id] && Array.isArray(this.detailedData[params.id])) {
            oldData = this.detailedData[params.id].slice(0);
          }

          // 删除原有的旧数据，在执行完所有处理之后再重新设置数据
          // 否则页面会无法刷新
          this.$delete(this.detailedData, params.id);

          // 取出表单元素：明细内元素的模板数据，并修改它们的id和name,防止重复
          for (let index = 0; index < params.childElements.length; index++) {
            // 临时数据
            let temp = Object.assign({}, params.childElements[index]);

            // 在id和name属性后新增一个唯一的随机数，以防重复
            let random1 = Math.random();
            let random2 = Math.random();
            if (random1 === random2) {
              random1 = random1 - 0.01;
            }

            temp.id = temp.id + "-" + random1.toString().split(".")[1];
            temp.name = temp.name + "-" + random2.toString().split(".")[1];

            if (oldData) {
              oldData[0][0].push(temp);
            } else {
              result[0][0].push(temp);
            }
          }

          // 设置新数据,以所有原始数据为一组,每次新增一组数据
          if (oldData) {
            this.$set(this.detailedData, params.id, oldData);
          } else {
            this.$set(this.detailedData, params.id, result);
          }
        }
      },

      // 减少 表单元素：明细 内的表单元素
      minusDetailElements: function (params) {
        if (this.mode === this.modeOption.renderMode.rendering) {
          // 旧的渲染数据
          let oldData;
          // 处理结果
          let result = this.formDataTemplate.slice(0);

          // 如果存在旧的渲染数据，就先将旧的数据独立出来保存
          if (this.detailedData[params.id] && Array.isArray(this.detailedData[params.id])) {
            oldData = this.detailedData[params.id].slice(0);
          }

          // 删除原有的旧数据，在执行完所有处理之后再重新设置数据
          // 否则页面会无法刷新
          this.$delete(this.detailedData, params.id);

          // 删除旧数据
          if (oldData) {
            let target = oldData[0][0];
            // 保留最后一组元素,作为原始数据,不完成删完
            if (target.length > params.childElements.length) {
              // 循环删除,每次都删除最后一个元素,有多少个原始表单元素就循环多少次
              // 达到每次删除都删除一组原始数据
              for (let index = 0; index < params.childElements.length; index++) {
                oldData[0][0].splice(target.length - 1, 1);
              }
            }
            this.$set(this.detailedData, params.id, oldData);
          }
        }
      },
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
            this.elementsData.push(this.formDataTemplate[0].slice(0));
          }
        } else if (newValue < oldValue) {
          let number = oldValue - newValue;
          this.elementsData.splice(-1, number);
        }
      },
      data: function (newValue, oldValue) {
        // for (let index1 = 0; index1 < newValue.length; index1++) {
        //   for (let index2 = 0; index2 < newValue[index1].length; index2++) {
        //     for (let index3 = 0; index3 < newValue[index2].length; index3++) {
        //       for (const key in newValue[index1][index2][index3]) {
        //         if (newValue[index1][index2][index3].hasOwnProperty(key)) {
        //           const element = newValue[index1][index2][index3][key];
        //           console.log(element)
        //         }
        //       }
        //     }
        //   }
        // }
        // !!!!!!!! 待解决：视图刷新成功，但数据传过来时为空
        this.elementsData = newValue;
      }
    },
  }
</script>
