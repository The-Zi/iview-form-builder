/*
 * @Author: The-Zi
 * @Date: 2018-07-04 17:11:32
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 14:24:14
 */
/*
 * @Author: The-Zi
 * @Date: 2018-07-03 16:38:15
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-04 17:10:06
 */
/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:34
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-06-28 18:06:01
 */


<template>
  <!-- 表单构建器：栅格系统 -->
  <Row :class="{
    'form-grid-wrap': true,
    'form-grid-wrap-mobile': nowDevice === devicesOption.mobile && nowMode !== modeOption.detailed,
  }">
    <Form ref="tempForm" :model="tempForm" label-position="top" inline>
      <!-- 表单行 -->
      <Col span="24" v-for="(row, rowIndex) in modulesData" :key="rowIndex"
      :class="{
        'form-grid-row': true,
        'form-grid-row-mobile': nowDevice === devicesOption.mobile && nowMode !== modeOption.detailed
      }">
        <!-- 表单列增减 -->
        <div class="form-grid-col-activer"
        v-if="nowMode === modeOption.builder && nowDevice === devicesOption.desktop">
          <!-- 刪除列 -->
          <Icon @click.native="gridColMinus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="删除列"
          type="minus"></Icon>

          <!-- 增加列 -->
          <Icon @click.native="gridColPlus(rowIndex)" class="activer" size="16" color="#a7a7a7" title="增加列"
          type="plus"></Icon>
        </div>
        <!-- 表单列 -->
        <Row class="form-grid-col-wrap">
          <Col :class="{'form-grid-col': true,
          'form-grid-col-mobile': nowMode !== modeOption.detailed && nowDevice === devicesOption.mobile}"
          :span="setColSize({rowIndex:rowIndex, nowIndex:colIndex})"
          v-for="(col, colIndex) in row" :key="colIndex">

            <!-- 拖动放置组件区域 -->
            <div v-activer class="form-grid-render-zoon" @dragover="allowDrop"
             @drop="drop({rowIndex: rowIndex,colIndex:colIndex}, $event)">

                <!-- =============== 提示信息 =============== -->
                <div :class="{'form-grid-col-tips': true, 'form-grid-col-tips-mobile': nowMode === modeOption.detailed}" v-if="!col || col.length <= 0">
                <!-- <p :class="{
                'form-grid-col-tips': nowMode === modeOption.detailed && nowDevice === devicesOption.mobile,
                'form-grid-col-tips-mobile' : nowDevice === devicesOption.mobile}" v-if="!col || col.length <= 0"> -->
                  <p>可拖入多个组件<br><span v-if="nowMode === modeOption.detailed">（不包含明细组件）</span></p>
                </div>

                <!-- =============== 表单组件渲染 =============== -->
                <formElements
                :mode="nowMode"
                :modulesData="{rowIndex: rowIndex,colIndex:colIndex, col: col}"
                @delete="delModule"
                @setting="editFormElement"
                @findCountFormula="saveCountFormula"
                @moneyFormat="moneyFormat"
                @count="performCount"
                @findCalculator="saveCalculator">
                </formElements>

                <!-- =============== 表单组件-明细 =============== -->
                <div v-if="nowMode !== modeOption.detailed" >
                  <div v-for="(modules, mIndex) in col" :key="mIndex">
                    <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.detailed.type"
                    :required="modules.require" :label="modules.label">
                        <!-- 操作组件 -->
                        <div class="form-modules-action" v-if="nowMode !== modeOption.detailed">
                          <!-- 设置 -->
                          <Icon class="action" size="16" color="#a7a7a7" title="设置"
                          @click.native="editFormElement({rowIndex: rowIndex, colIndex: colIndex,
                          mIndex: mIndex, module: modules})"
                          type="ios-gear"/>
                          <!-- 删除 -->
                          <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                          @click.native="delModule({rowIndex: rowIndex, colIndex: colIndex, mIndex: mIndex,
                          module: modules})"/>
                        </div>

                        <!-- 设置 表单组件-明细 的渲染数据 -->
                        <!-- {{setDetailRenderData(modules)}} -->

                        <!-- ========== 表单组件-明细 v2.0.0：每次自增都新增一个明细组件 ========== -->
                        <!-- 通过递归当前组件，实现表单组件-明细 -->
                        <!-- <iview-form-builder-grid
                        :mode="modeOption.detailed"
                        :device="nowDevice"
                        :data="handleDetailData(modules)"
                        @addModuels="addDetailElement(modules, $event)"
                        @setModuels="setDetailElement(modules, $event)"
                        @delModuels="delDetailElement(modules, $event)">
                        </iview-form-builder-grid> -->

                        <!-- 表单组件-明细，操作 -->
                        <!-- <div class="form-modules-detailed-action" v-if="mIndex === col.length - 1">
                          <Button type="primary" icon="plus"
                          @click.native="plusDetail({module: modules, col: col})">
                            {{modules.actionPlus}}
                          </Button>
                        </div> -->

                        <!-- ========== 表单组件-明细 v1.0.0：在同一明细内自增原始模板数据 ========== -->
                        <iview-form-builder-grid
                        :mode="modeOption.detailed"
                        :device="nowDevice"
                        :data="handleDetailData(modules)"
                        :calculator="calculatorList"
                        :countFormula="countFormulaList"
                        @count="performCount"
                        @findCalculator="addCalculatorList(modules, $event)"
                        @delCalculator="delCalculatorList(modules, $event)"
                        @addModuels="addDetailElement(modules, $event)"
                        @setModuels="setDetailElement(modules, $event)"
                        @delModuels="delDetailElement(modules, $event)"
                        @findCountFormula="saveCountFormula">
                        </iview-form-builder-grid>

                        <!-- 增减明细 -->
                        <div class="form-modules-detailed-action">
                            <Button type="primary" icon="plus" long @click.native="plusDetailModuels(modules)">
                              {{modules.actionPlus}}
                            </Button>
                            <!-- <ButtonGroup>
                                <Button type="ghost" icon="minus" @click.native="minusDetailModuels(modules)">
                                  {{modules.actionMinus}}
                                </Button>
                                <Button type="ghost" icon="plus" @click.native="plusDetailModuels(modules)">
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

    <!-- =============== 栅格系统：增减行数 =============== -->
    <Col class="form-grid-row-action" span="24"
    v-if="nowMode === modeOption.builder && nowDevice !== devicesOption.mobile">
      <ButtonGroup shape="circle">
          <Button type="default" @click.native="rowMinus" title="删除行">
            &nbsp;
            &nbsp;
            &nbsp;
            <Icon type="minus"></Icon>
            &nbsp;
            &nbsp;
            &nbsp;
          </Button>
          <Button type="default" @click.native="rowPlus" title="增加行">
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

    <!-- =============== 对话框：表单组件设置 =============== -->
    <Modal v-model="modulesSetingModalToogle" :closable="false" :mask-closable="false">
      <!-- 标题 -->
      <p slot="header" class="congfig-modal-header">
        <span>{{moduleConfig.title}}设置</span>
      </p>

      <!-- 表单组件配置项 & 计算公式配置项-->
      <!-- 在同一对话框切换显示表单组件配置项和计算公式配置项 -->
      <Form v-show="!formulasSettingToogle" ref="configForm" :rules="ruleValidate" :model="moduleConfig" label-position="left"
       :label-width="100">

        <!-- 设为必填项 -->
        <!-- 表单组件：说明文本 和 明细，不需要必填项设置 -->
        <FormItem v-if="moduleConfig.type !== module.textRead.type &&
        moduleConfig.type !== module.detailed.type" label="设为必填项：">
          <Checkbox v-model="moduleConfig.require"></Checkbox>
        </FormItem>

        <!-- 字段名 -->
        <!-- 表单组件： 明细，不需要字段名设置 -->
        <FormItem label="字段名：" v-if="moduleConfig.type !== module.detailed.type" prop="name">
          <Input v-model="moduleConfig.name" @keyup.native="filterBlank(moduleConfig)"
          placeholder="只能输入英文"></Input>
        </FormItem>

        <!-- 标题 -->
        <FormItem label="标题：" prop="label">
          <Input v-model="moduleConfig.label" @keyup.native="filtersLabel(moduleConfig)"
          :maxlength="20" placeholder="请输入该组件的名称，不能输入加减乘除和小数点"></Input>
        </FormItem>

        <!-- 文本输入框配置 -->
        <template v-if="moduleConfig.type === module.inputText.type">
          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>

          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>
        </template>

        <!-- 数字输入框配置 -->
        <template v-if="moduleConfig.type === module.inputNumber.type">
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
        <template v-if="moduleConfig.type === module.count.type">
          <FormItem label="计算公式：">
            <!-- 当前的计算公式 -->
            <Input v-model="moduleConfig.nowFormulas" placeholder="请输入计算公式 " :readonly="true"
            @click.native="editFormulasModal">
            </Input>
          </FormItem>

          <FormItem label="计算结果：">
            <Select v-model="moduleConfig.rounding">
              <Option v-for="(item, index) in moduleConfig.roundingOption"
              :value="item.value" :key="index">
              {{item.label}}
              </Option>
            </Select>
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
        <template v-if="moduleConfig.type === module.money.type">
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
        <template v-if="moduleConfig.type === module.radio.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 多选配置 -->
        <template v-if="moduleConfig.type === module.checkbox.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.configValue" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 下拉选择配置 -->
        <template v-if="moduleConfig.type === module.select.type">
          <FormItem label="默认值：">
             <Input v-model="moduleConfig.value" placeholder="默认值"></Input>
          </FormItem>

          <FormItem label="选项编辑：">
            <Input v-model="moduleConfig.editList" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 多行文本配置 -->
        <template v-if="moduleConfig.type === module.textarea.type">
          <FormItem label="默认值：">
            <Input v-model="moduleConfig.value" type="textarea" :rows="4"></Input>
          </FormItem>

          <FormItem label="提示信息：" prop="tips">
            <Input v-model="moduleConfig.tips" placeholder="最大长度50" :maxlength="50"></Input>
          </FormItem>
        </template>

        <!-- 说明文本配置 -->
        <template v-if="moduleConfig.type === module.textRead.type">
          <FormItem label="默认值：">
            <Input v-model="moduleConfig.value" type="textarea" :rows="4"></Input>
          </FormItem>
        </template>

        <!-- 时间日期 与 时间日期范围 配置 -->
        <template v-if="moduleConfig.type === module.dateTime.type ||
        moduleConfig.type === module.dateTimeRange.type">
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
        <template v-if="moduleConfig.type === module.detailed.type">
          <FormItem label="名称2（增加）：" prop="plus">
            <Input v-model="moduleConfig.actionPlus" placeholder="增加明细的动作名称"></Input>
          </FormItem>
          <FormItem label="名称1（减少）：" prop="minus">
            <Input v-model="moduleConfig.actionMinus" placeholder=""></Input>
          </FormItem>
        </template>
      </Form>

      <!-- 对话框底部 -->
      <div slot="footer" v-show="!formulasSettingToogle">
        <Button type="ghost" size="large" @click.native="cancelConfigModal">关闭</Button>
        <Button type="primary" size="large" @click.native="saveModuleConfig">确定</Button>
      </div>

      <!-- 配置计算公式 -->
      <template v-show="formulasSettingToogle">
        <div class="config-formulas-wrap" v-show="formulasSettingToogle">
          <!-- 计算公式 -->
          <Row class="config-formulas-item">
            <Col span="24">
              <!-- 错误提示 -->
              <p style="color:red;" v-if="!moduleConfig.formulasValid">编辑的公式不符合计算法则，无法计算</p>
              <!-- 计算公式显示框 -->
              <div class="formulas-show-zoon">
                <p v-html="moduleConfig.tempFormulas"></p>
              </div>

              <div style="text-align: right;">
                <Icon class="formulas-minus" type="backspace-outline" @click.native="delTempCalculator"></Icon>
                <span class="formulas-clear" @click="delTempCalculator('all')">清空</span>
              </div>
              <!-- <Input v-model="moduleConfig.tempFormulas" :readonly="true"
              placeholder="请在下方选择计算对象和计算符号">
                <Button slot="append" icon="backspace-outline" @click.native="delTempCalculator"></Button>
                <Button slot="append" @click.native="delTempCalculator('all')">清空</Button>
              </Input> -->
            </Col>
          </Row>

          <!-- 提示 -->
          <Row class="config-formulas-item">
            <Col span="24">
              <p>计算公式可用来完成数字输入框数据的自动计算，例如：采购单内设置计算公式“合计=单价×数量”，
              发起人填写单价、数量后，组件将自动计算出合计金额，无需手动计算。</p>
            </Col>
          </Row>

          <!-- 可计算对象 -->
          <Row class="config-formulas-item">
            <Col span="4">
              <p><Icon type="information-circled" :size="14"></Icon>&nbsp;计算对象：&nbsp;</p>
            </Col>

            <Col span="20">
              <template  v-for="(item, index) in nowCalculatorList">
                <Button v-if="item.id !== moduleConfig.id" size="small"  @click.native="editFormulas(item)"
                :key="index" style="margin:0 5px 2px 0;">
                  {{item.label}}
                </Button>
              </template>
              <!-- 通过将nowCalculatorList转成JSON格式的字符串判断其是否为空对象 -->
              <p v-if="JSON.stringify(nowCalculatorList) === '{}'">
                没有可计算的对象，请返回表单添加
              </p>
            </Col>
          </Row>

          <!-- 计算符号 -->
          <Row class="config-formulas-item">
            <Col span="4">
              <p></Icon>&nbsp;计算符号：&nbsp;</p>
            </Col>

            <Col span="20">
              <Button type="ghost" size="default" @click.native="editFormulas('+')">+</Button>
              <Button type="ghost" size="default" @click.native="editFormulas('-')">-</Button>
              <Button type="ghost" size="default" @click.native="editFormulas('×')">×</Button>
              <Button type="ghost" size="default" @click.native="editFormulas('÷')">÷</Button>
              <Button type="ghost" size="default" @click.native="editFormulas('(')">(</Button>
              <Button type="ghost" size="default" @click.native="editFormulas(')')">)</Button>
            </Col>
          </Row>

          <!-- 数字键盘 -->
          <Row class="config-formulas-item number-keyboard">
            <Col span="4">
              <p>&nbsp;数字键盘：&nbsp;</p>
            </Col>

            <Col span="20">
              <p>
                <Button type="ghost" size="default" @click.native="editFormulas('1')">1</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('2')">2</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('3')">3</Button>
              </p>
              <p>
                <Button type="ghost" size="default" @click.native="editFormulas('4')">4</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('5')">5</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('6')">6</Button>
              </p>
              <p>
                <Button type="ghost" size="default" @click.native="editFormulas('7')">7</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('8')">8</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('9')">9</Button>
              </p>
              <p>
                <Button type="ghost" size="default" @click.native="editFormulas('0')">0</Button>
                <Button type="ghost" size="default" @click.native="editFormulas('.')">&nbsp;.</Button>
              </p>
            </Col>
          </Row>
        </div>

        <!-- 取消/保存 计算公式配置 -->
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
@import "../styles/grid";
@import "../styles/configModal";
</style>


<script>
// =============== 导入模块 ===============
// 配置文件
import config from "../config";
// 表单组件
import formElements from "./elements";

export default {
  // 组件名
  name: "iview-form-builder-grid",

  // 组件
  components: {
    formElements
  },

  // 自定义属性
  props: {
    // 组件模式
    mode: {
      default: config.mode.builder
    },

    // 显示设备
    device: {
      default: config.devices.desktop
    },

    // 渲染用表单数据
    renderData: {
      default: Array
    },

    // 计算对象
    calculator: {
      default: Object
    },

    // 计算公式
    countFormula: {
      default: Object
    },

    // 生命周期钩子：在删除组件前
    beforDeleteModule: {
      default: Function
    }
  },

  // 自定义指令
  directives: {
    // 活动中
    activer: {
      inserted: el => {
        // 添加提示色
        let activerOn = $event => {
          el.style.backgroundColor = "#e6e6e6";
          // 阻止冒泡和默认行为，
          // 防止以 表单组件-明细 形式调用的时候和父组件的事件冲突
          $event.preventDefault();
          $event.stopPropagation();
        };

        // 清除提示色
        let activerOff = $event => {
          el.style.backgroundColor = "#fff";
          // 阻止冒泡和默认行为，
          // 防止以 表单组件-明细 形式调用的时候和父组件的事件冲突
          $event.preventDefault();
          $event.stopPropagation();
        };

        // 绑定事件
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
      // 设备选项
      devicesOption: config.devices,
      // 组件模式选项
      modeOption: config.mode,
      // 表单组件配置
      module: config.formElement,
      moduleGroups: config.formElementGroups,
      // 表单组件数据，是一个三维数组
      // 第一维是整个表单结构
      // 第二维是表单布局结构中的行
      // 第三维是表单布局结构中行里面的列
      // 第三维内的是表单组件数据对象
      modulesData: [[[]]],
      // 表单行数
      rowNumber: 1,
      // 临时表单数据，占位用，防止报错
      tempForm: {},
      // 表单组件设置对话框
      modulesSetingModalToogle: false,
      // 计算公式 配置
      formulasSettingToogle: false,
      // 表单组件配置
      moduleConfig: {},
      // 暂存当前配置中的组件
      targetModule: "",
      // 表单组件配置验证规则
      ruleValidate: {
        name: [
          {
            required: true,
            message: "必填，只能输入英文字母和数字，并只能以英文字母开头",
            trigger: "blur",
            pattern: /[0-9A-Za-z]/
          }
        ]
      },
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
      calculatorMark:
        'data-formulas="bukegenggaiyongyupanduanjisuanduixiangdebaijifuhao"',
      // 明细组件内计算公式
      detailedCountFoumula: {}
    };
  },

  // 计算属性
  computed: {
    // 当前模式
    nowMode: function() {
      return this.mode;
    },
    // 当前模式
    nowDevice: function() {
      return this.device;
    },
    // 当前的可计算对象
    nowCalculatorList: function() {
      if (this.calculator && typeof this.calculator === "object") {
        return this.calculator;
      } else {
        return this.calculatorList;
      }
    },
    // 可计算对象列表
    calculatorList: {
      get: function() {
        return this.calculator;
      },
      set: function(newValue) {
        this.calculator = Object.assign({}, this.calculator, newValue);
      }
    },
    // 计算公式组件列表
    countFormulaList: {
      get: function() {
        return this.countFormula;
      },
      set: function(newValue) {
        this.countFormula = Object.assign({}, this.countFormula, newValue);
      }
    }
  },

  // 方法
  methods: {
    // 深克隆
    deepClone: function(obj) {
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

    // 获取表单数据模板
    getFormDataTemplate: function(params) {
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

    // 阻止被放置数据的组件的默认行为
    allowDrop: function(ev) {
      ev.preventDefault();
    },

    // 拖放添加表单组件
    drop: function(params, $event) {
      // 获取拖动传递过来的值
      let module = $event.dataTransfer.getData("Text");
      // 生成时间戳，用于区分不同的字段名
      let tempDate = new Date();
      // 临时表单组件项
      let tempItem = {};

      // 表单组件类型
      tempItem.type = module;
      // 唯一识别ID
      tempItem.id = module + "-" + tempDate.getTime();
      // 表单组件值
      tempItem.value = "";
      // 该表单组件是否为必填项
      tempItem.require = false;
      // 默认表单key
      tempItem.name = module + tempDate.getTime();

      // 配置并添加表单组件
      switch (module) {
        // 预算
        case this.moduleGroups.budget.type:
          tempItem.title = this.moduleGroups.budget.title;
          tempItem.label = tempItem.title + "：";
          tempItem.value = "";
          tempItem.name = "is_excess_budget";
          tempItem.require = true;
          tempItem.itemList = [0, 1];
          break;

        // 文本输入框
        case this.module.inputText.type:
          tempItem.title = this.module.inputText.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          // 表单组件提示信息/占位符
          tempItem.tips = "";
          break;

        // 数字输入框
        case this.module.inputNumber.type:
          tempItem.title = this.module.inputNumber.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          // 表单组件提示信息/占位符
          tempItem.tips = "";
          tempItem.value = null;
          tempItem.max = 20000;
          tempItem.min = 0;
          break;

        // 计算公式
        case this.module.count.type:
          tempItem.title = this.module.count.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          // 表单组件提示信息/占位符
          tempItem.tips = "";
          tempItem.value = null;
          // 有效的计算公式，编辑完成后最后保存用的计算公式
          tempItem.validFormulas = "";
          // 当前计算公式，用于在计算公式组件配置界面显示，由数学计算符号和可计算对象的label组成
          tempItem.nowFormulas = "";
          // 临时计算公式,用于编辑计算公式时使用，由数学计算符号和用html显示的可计算对象的label组成
          tempItem.tempFormulas = "";
          // 计算用计算公式，用于执行计算，由数学计算符号和可计算对象的id组成
          tempItem.countFormulas = "";
          // 临时计算公式用于
          tempItem.tempCountFormulas = "";
          // 当前计算公式组件的计算公式有效性
          tempItem.formulasValid = true;

          tempItem.resultTips = "大写：";
          tempItem.unit = "";
          tempItem.cnNum = "";
          tempItem.showUpper = true;
          tempItem.rounding = 0;
          tempItem.roundingOption = [
            {
              value: 0,
              label: "取整"
            },
            {
              value: 1,
              label: "保留小数点后 1 位数"
            },
            {
              value: 2,
              label: "保留小数点后 2 位数"
            },
            {
              value: 3,
              label: "保留小数点后 3 位数"
            },
            {
              value: 4,
              label: "保留小数点后 4 位数"
            }
          ];
          break;

        // 金额
        case this.module.money.type:
          tempItem.title = this.module.money.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          // 表单组件提示信息/占位符
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
        case this.module.radio.type:
          tempItem.title = this.module.radio.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.value = "";
          tempItem.itemList = ["选项1", "选项2", "选项3"];
          break;

        // 复选框
        case this.module.checkbox.type:
          tempItem.title = this.module.checkbox.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.value = [];
          tempItem.itemList = [
            "选项1",
            "选项2",
            "选项3",
            "选项4",
            "选项5",
            "选项6",
            "选项7",
            "选项8"
          ];
          break;

        // 下拉框
        case this.module.select.type:
          tempItem.title = this.module.select.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.value = "";
          tempItem.itemList = ["选项1", "选项2", "选项3"];
          break;

        // 多行文本
        case this.module.textarea.type:
          tempItem.title = this.module.textarea.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.tips = "";
          break;

        // 说明信息
        case this.module.textRead.type:
          tempItem.title = this.module.textRead.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.value = "说明信息";
          break;

        // 时间日期
        case this.module.dateTime.type:
          tempItem.title = this.module.dateTime.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.tips = "";
          tempItem.format = 1;
          break;

        // 时间日期范围
        case this.module.dateTimeRange.type:
          tempItem.title = this.module.dateTimeRange.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);

          tempItem.tips = "";
          tempItem.format = 1;
          break;

        // 明细
        case this.module.detailed.type:
          tempItem.title = this.module.detailed.title;

          // 在表单组件label后面加上数量
          // 要先设置title属性
          tempItem.label = this.countModuelsNum(tempItem);
          // 表单组件-明细：渲染用模板
          tempItem.renderTemplate = [];
          tempItem.actionPlus = "增加";
          tempItem.actionMinus = "减少";
          // 测试用默认数据
          // let x = '[[[{"type":"count","id":"count-1532399028599","value":null,"require":false,"name":"count1532399028599","title":"计算公式","label":"总价","tips":"","validFormulas":"   单价 × 数量","nowFormulas":"总价 =   单价 × 数量","tempFormulas":"总价 =    <span data-formulas=\"bukegenggaiyongyupanduanjisuanduixiangdebaijifuhao\" style=\"display: inline-block;padding: 0 2px;margin: 2px 0;border-radius: 3px; padding: 2px 7px;border: 1px solid #dddee1;font-size: 12px; background-color: #f7f7f7;\">单价</span> × <span data-formulas=\"bukegenggaiyongyupanduanjisuanduixiangdebaijifuhao\" style=\"display: inline-block;padding: 0 2px;margin: 2px 0;border-radius: 3px; padding: 2px 7px;border: 1px solid #dddee1;font-size: 12px; background-color: #f7f7f7;\">数量</span>","countFormulas":"&nbsp;inputNumber-1532399056031&nbsp;×&nbsp;inputNumber-1532399096338","tempCountFormulas":"&nbsp;inputNumber-1532399056031&nbsp;×&nbsp;inputNumber-1532399096338","formulasValid":true,"resultTips":"大写：","unit":"","cnNum":"","showUpper":true,"rounding":0,"roundingOption":[{"value":0,"label":"取整"},{"value":1,"label":"保留小数点后 1 位数"},{"value":2,"label":"保留小数点后 2 位数"},{"value":3,"label":"保留小数点后 3 位数"},{"value":4,"label":"保留小数点后 4 位数"}]},{"type":"detailed","id":"detailed-1532399038823","value":"","require":false,"name":"detailed1532399038823","title":"明细","label":"","renderTemplate":[{"type":"inputNumber","id":"inputNumber-1532399056031","value":null,"require":false,"name":"inputNumber1532399056031","title":"数字输入框","label":"单价","tips":"","max":20000,"min":0},{"type":"inputNumber","id":"inputNumber-1532399096338","value":null,"require":false,"name":"inputNumber1532399096338","title":"数字输入框","label":"数量","tips":"","max":20000,"min":0}],"actionPlus":"增加","actionMinus":"减少"}]]]';
          // tempItem.renderTemplate[0] = JSON.parse(x)[0][0][0];
          // tempItem.renderTemplate[1] = JSON.parse(x)[0][0][1];
          // tempItem.renderTemplate[2] = JSON.parse(x)[0][0][2];
          break;
      }

      // 判断是否允许添加表单组件
      if (this.nowMode === this.modeOption.detailed) {
        // 不允许在表单组件-明细，内添加表单组件-明细
        if (tempItem.type !== this.module.detailed.type) {
          // 传递新表单组件的数据到父组件
          if (this.nowMode === this.modeOption.detailed) {
            // 自定事件：添加组件事件，以对象形式传递新添加的表单组件
            this.$emit("addModuels", tempItem);
          }

          // 添加新表单组件
          this.modulesData[params.rowIndex][params.colIndex].push(tempItem);
        }
      } else {
        // 添加新表单组件
        this.modulesData[params.rowIndex][params.colIndex].push(tempItem);
      }

      // 阻止默认行为和事件冒泡，防止触发表单构建器的拖放事件
      $event.preventDefault();
      $event.stopPropagation();
    },

    // 计算各表单组件数量
    countModuelsNum: function(params) {
      if (this.modulesIndex[params.type] !== undefined) {
        this.modulesIndex[params.type] = this.modulesIndex[params.type] + 1;
        this.modulesLength[params.type] = this.modulesLength[params.type] + 1;
      } else {
        this.modulesIndex[params.type] = 0;
        this.modulesLength[params.type] = 0;
      }

      return (
        params.title +
        (this.modulesIndex[params.type] === 0
          ? ""
          : "(" + this.modulesIndex[params.type] + ")")
      );
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
        let zeroCount = 0;
        let IntLen = integerNum.length;

        for (let i = 0; i < IntLen; i++) {
          let n = integerNum.substr(i, 1);
          let p = IntLen - i - 1;
          let q = p / 4;
          let m = p % 4;

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

    // 计算公式编辑对话框
    editFormulasModal: function() {
      // 重置编辑用临时计算公式
      this.moduleConfig.tempFormulas = "";

      // 预设编辑用临时计算公式为当前的计算公式
      this.moduleConfig.tempFormulas = this.moduleConfig.label + " " + "=";

      // 如果存在计算公式则对计算公式进行格式化处理
      let formulas = this.moduleConfig.nowFormulas.split("=")[1].split(" ");

      // 将当前的计算公式解析成编辑用的计算公式样式
      for (let index = 0; index < formulas.length; index++) {
        if (formulas[index].match(/[0-9\.\+\-\×\÷\(\)\s]/g)) {
          this.moduleConfig.tempFormulas =
            this.moduleConfig.tempFormulas + " " + formulas[index];
        } else {
          this.moduleConfig.tempFormulas =
            this.moduleConfig.tempFormulas +
            " " +
            this.formulasStyle(formulas[index]);
        }
      }

      // 重置计算公式的有效性检查结果为正确
      this.moduleConfig.formulasValid = true;

      // 显示计算公式配置
      this.formulasSettingToogle = true;
    },

    // 计算公式编辑-隐藏对话框
    closeFormulasSetting: function() {
      // 重置当前显示用计算公式
      this.moduleConfig.nowFormulas =
        this.moduleConfig.label + " = " + this.moduleConfig.validFormulas;

      // 关闭计算公式编辑对话框
      this.formulasSettingToogle = false;
    },

    // 计算公式编辑-保存
    saveFormulasSetting: function() {
      // 将需要保持的计算公式中的计算对象名称全部替换成10
      let countStatus;
      let countFormulas = "";
      // let count = this.moduleConfig.tempCountFormulas.replace(/([^\+\-\×\÷\(\)\.\d]+)/g,"10");
      // 重置验证状态
      this.moduleConfig.formulasValid = true;
      // 判断是否
      if (
        this.moduleConfig.tempFormulas.match(
          "</span> <span " + this.calculatorMark
        )
      ) {
        this.moduleConfig.formulasValid = false;
        return;
      }

      // 格式化计算公式为计算机可以识别的形式用于判断计算公式是否正确
      let tempFormulas = this.moduleConfig.tempCountFormulas.split("&nbsp;");
      for (let index = 0; index < tempFormulas.length; index++) {
        if (
          typeof Number(tempFormulas[index]) === "number" &&
          Number(tempFormulas[index]) === Number(tempFormulas[index])
        ) {
          countFormulas = countFormulas + tempFormulas[index];
        } else if (tempFormulas[index].search(/[\.\+\-\×\÷\(\)]/g) === 0) {
          countFormulas = countFormulas + tempFormulas[index];
        } else {
          countFormulas = countFormulas + 10;
        }
      }

      countFormulas = countFormulas.replace(/\×/g, "*");
      countFormulas = countFormulas.replace(/\÷/g, "/");

      // 执行替换处理后的计算公式，判断是否为有效的计算公式
      try {
        eval(countFormulas);
        countStatus = true;
        this.moduleConfig.formulasValid = true;
      } catch (error) {
        countStatus = false;
        this.moduleConfig.formulasValid = false;
      }

      // 只有当计算公式为有效时，才能保存
      if (countStatus === true) {
        // 关闭对话框
        this.formulasSettingToogle = false;
        // 不是有效的计算公式
      } else {
        this.moduleConfig.formulasValid = false;
      }
    },

    // 删除计算对象
    delTempCalculator: function(params) {
      // 一次性清空所有
      if (params === "all") {
        // 临时编辑用计算公式
        this.moduleConfig.tempFormulas = this.moduleConfig.label + " " + "=";

        // 同时清空当前显示用的计算公式
        this.moduleConfig.nowFormulas = this.moduleConfig.label + " " + "=";

        // 临时计算用计算公式
        this.moduleConfig.tempCountFormulas = "";
        // 一次删除一个
      } else {
        // 取出计算公式
        let formulas = this.moduleConfig.nowFormulas.split("=");
        let targetFormulas = formulas[1].split(" ");
        targetFormulas.splice(targetFormulas.length - 1, 1);

        this.moduleConfig.nowFormulas =
          this.moduleConfig.label + " " + "=" + " " + targetFormulas.join(" ");

        // 重置编辑用临时计算公式
        this.moduleConfig.tempFormulas = this.moduleConfig.label + " " + "=";

        // 更新删除后的编辑用临时计算公式
        for (let index = 0; index < targetFormulas.length; index++) {
          if (targetFormulas[index].match(/[0-9\.\+\-\×\÷\(\)]/g)) {
            this.moduleConfig.tempFormulas =
              this.moduleConfig.tempFormulas + " " + targetFormulas[index];
          } else {
            this.moduleConfig.tempFormulas =
              this.moduleConfig.tempFormulas +
              " " +
              this.formulasStyle(targetFormulas[index]);
          }
        }
      }
    },

    // 编辑计算公式
    editFormulas: function(params) {
      if (typeof params === "object") {
        // 临时计算公式,在计算公式编辑框内显示，用于编辑计算公式时使用，
        // 由数学计算符号和用html显示的可计算对象的label组成
        this.moduleConfig.tempFormulas =
          this.moduleConfig.tempFormulas +
          " " +
          this.formulasStyle(params.label);

        // 同时更新当前显示用的计算公式
        this.moduleConfig.nowFormulas =
          this.moduleConfig.nowFormulas + " " + params.label;

        // 临时计算用计算公式，用于执行实际的计算
        this.moduleConfig.tempCountFormulas =
          this.moduleConfig.tempCountFormulas + "&nbsp;" + params.id;
      } else if (typeof params === "string") {
        // 临时计算公式,在计算公式编辑框内显示，用于编辑计算公式时使用，
        // 由数学计算符号和用html显示的可计算对象的label组成
        this.moduleConfig.tempFormulas =
          this.moduleConfig.tempFormulas + " " + params;

        // 同时更新当前显示用的计算公式
        this.moduleConfig.nowFormulas =
          this.moduleConfig.nowFormulas + " " + params;

        // 临时计算用计算公式，用于执行实际的计算
        this.moduleConfig.tempCountFormulas =
          this.moduleConfig.tempCountFormulas + "&nbsp;" + params;
      }
    },

    // 执行计算
    performCount: function(params, source) {
      // 自定义事件：计算
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
                let rules = new RegExp(targetCalculator.id, "g");

                // 对来自表单组件：明细，的计算对象的id进行处理，以获取其在计算公式配置中对应的id
                if (
                  source === this.modeOption.detailed &&
                  targetCalculator.id === params.id
                ) {
                  let targetID =
                    params.id.split("-")[0] + "-" + params.id.split("-")[1];
                  rules = new RegExp(targetID, "g");
                }

                // 将值替换进计算公式中对应的位置
                if (formulaResult) {
                  formulaResult = formulaResult.replace(
                    rules,
                    targetCalculator.value
                  );
                } else {
                  formulaResult = targetFormula.countFormulas.replace(
                    rules,
                    targetCalculator.value
                  );
                }

                // 替换计算公式结果中的乘除计算符号
                formulaResult = formulaResult.replace(/[×]/, "*");
                formulaResult = formulaResult.replace(/[÷]/, "/");
              }
            }

            // 执行计算
            try {
              // 根据计算公式计算结果
              let countResult = eval(
                formulaResult
                  .toString()
                  .split("&nbsp;")
                  .join("")
              );

              // 如果计算对象来自表单组件：明细,并且之前执行的计算不包括该计算对象，则新的计算结果与旧的计算结果相加
              if (
                source === this.modeOption.detailed &&
                !this.detailedCountFoumula[params.id]
              ) {
                targetFormula.value =
                  Number(targetFormula.value) +
                  Number(countResult.toFixed(targetFormula.rounding));

                // 保存当前计算对象用于下次计算时判断是否其来源以及是否需要与旧结果相加
                this.detailedCountFoumula[params.id] = {
                  id: params.id,
                  value: typeof params.value === "null" ? 0 : params.value
                };

                // 普通情况下的计算 与 来自于表单组件：明细，但之前执行的计算包括该计算对象
              } else {
                targetFormula.value = Number(
                  countResult.toFixed(targetFormula.rounding)
                );
              }

              // 将计算结果转换成中文大写数字金额
              this.moneyFormat(targetFormula);
            } catch (error) {
              console.log(error);
            }
          }
        }
      }
    },

    // 计算公式样式设置
    formulasStyle: function(params) {
      if (!params) {
        return "";
      }
      // 返回编辑用临时计算公式
      return (
        '<span ' +
        this.calculatorMark +
        ' style="display: inline-block;padding: 0 2px;margin: 2px 0;' +
        'border-radius: 3px; padding: 2px 7px;border: 1px solid #dddee1;' +
        'font-size: 12px; background-color: #f7f7f7;">' +
        params +
        '</span>'
      );
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

    // 设置表单组件-明细 对应的渲染数据
    setDetailRenderData: function(params) {
      this.detailedRenderData[params.id] = this.getFormDataTemplate();
    },

    // 表单组件-明细: 将添加的可计算对象保存到统一的可计算对象列表中
    addCalculatorList: function(params, $event) {
      this.calculatorList[$event.id] = $event;
    },

    // 表单组件-明细: 删除可计算对象列表中的指定对象
    delCalculatorList: function(params, $event) {
      this.$delete(this.calculatorList, params.id);
    },

    // 表单组件-明细，数据处理
    handleDetailData: function(params) {
      // 与每个表单组件-明细相对应的渲染数据
      // if (!this.detailedRenderData[params.id]) {
      // this.detailedRenderData[params.id] = this.getFormDataTemplate();
      // }
      this.detailedRenderData[params.id] = this.getFormDataTemplate();

      // 根据原始模板数据设置渲染数据
      if (
        params.renderTemplate.length > 0 &&
        this.detailedRenderData[params.id][0][0].length === 0
      ) {
        for (let index = 0; index < params.renderTemplate.length; index++) {
          let temp = Object.assign({}, params.renderTemplate[index]);
          this.detailedRenderData[params.id][0][0].push(temp);
        }
      }

      return this.detailedRenderData[params.id];
    },

    // 表单组件-明细，添加表单组件
    addDetailElement: function(params, $event) {
      // 原始模板数据
      params.renderTemplate.push($event);

      // 渲染数据
      this.detailedRenderData[params.id][0][0].push($event);
    },

    // 表单组件-明细，修改表单组件配置
    setDetailElement: function(params, $event) {
      for (let index = 0; index < params.renderTemplate.length; index++) {
        if (params.renderTemplate[index].id === $event.id) {
          params.renderTemplate[index] = Object.assign(
            {},
            params.renderTemplate[index],
            $event
          );
        }
      }
    },

    // 表单组件-明细，删除表单组件
    delDetailElement: function(params, $event) {
      // 删除 原始模板数据
      for (let index = 0; index < params.renderTemplate.length; index++) {
        if (params.renderTemplate[index].id === $event.id) {
          params.renderTemplate.splice(index, 1);
        }
      }

      // 删除渲染数据
      // for (let index = 0; index < this.detailedRenderData[params.id][0][0].length; index++) {
      //   if (this.detailedRenderData[params.id][0][0][index].id === $event.id) {
      //     this.detailedRenderData[params.id][0][0].splice(index, 1);
      //   }
      // }
    },

    // 表单组件-明细 v1.0.0 自增明细内原始模板组件
    plusDetailModuels: function(params) {
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

    // 表单组件-明细 v2.0.0 自增整个明细
    plusDetail: function(params) {
      let result = {};

      for (const key in params.module) {
        if (params.module.hasOwnProperty(key)) {
          if (typeof params.module[key] === "Object") {
            result[key] = Object.assign({}, params.module[key]);
          } else {
            result[key] = params.module[key];
          }
        }
      }

      // 在id和name属性后新增一个唯一的随机数，以防重复
      let random1 = Math.random();
      let random2 = Math.random();

      if (random1 === random2) {
        random1 = random1 - 0.01;
      }

      result.id = result.id + "-" + random1.toString().split(".")[1];
      result.name = result.name + "-" + random2.toString().split(".")[1];
      this.detailedRenderData[result.id] = this.getFormDataTemplate();

      this.plusDetailModuels(params.module);
      params.col.push(result);
    },

    // 表单组件-明细 自减明细内原始模板组件
    minusDetailModuels: function(params) {
      if (this.nowMode === this.modeOption.render) {
        // 已有或默认的渲染数据
        let target = this.detailedRenderData[params.id][0][0];

        // 保留最后一组组件,作为原始数据,不完成删完
        if (target.length > params.renderTemplate.length) {
          // 循环删除,每次都删除最后一个组件,有多少个原始表单组件就循环多少次
          // 达到每次删除都删除一组原始数据
          for (let index = 0; index < params.renderTemplate.length; index++) {
            target.splice(target.length - 1, 1);
          }
        }

        // // 设置新数据
        // this.$set(this.detailedRenderData, params.id, oldData);

        // // 旧的渲染数据
        // let oldData;
        // // 处理结果
        // let result = this.getFormDataTemplate();

        // // 如果存在旧的渲染数据，就先将旧的数据独立出来保存
        // if (this.detailedRenderData[params.id] && this.detailedRenderData[params.id][0][0].length > 0) {
        //   oldData = this.detailedRenderData[params.id].slice(0);
        // }

        // // 删除原有的旧数据，在执行完所有处理之后再重新设置数据
        // // 否则页面会无法刷新
        // this.$delete(this.detailedRenderData, params.id);

        // // 删除旧数据
        // if (oldData) {
        //   let target = oldData[0][0];
        //   // 保留最后一组组件,作为原始数据,不完成删完
        //   if (target.length > params.renderTemplate.length) {
        //     // 循环删除,每次都删除最后一个组件,有多少个原始表单组件就循环多少次
        //     // 达到每次删除都删除一组原始数据
        //     for (let index = 0; index < params.renderTemplate.length; index++) {
        //       oldData[0][0].splice(target.length - 1, 1);
        //     }
        //   }
        //   // 设置新数据
        //   this.$set(this.detailedRenderData, params.id, oldData);
        // }
      }
    },

    // 组件设置
    editFormElement: function(params) {
      let row = params.rowIndex;
      let col = params.colIndex;
      let m = params.mIndex;
      let module = this.modulesData[row][col][m];

      // 组件配置变量
      this.moduleConfig = {};
      this.moduleConfig = Object.assign({}, this.moduleConfig, module);

      // 单选框组件、多选框组件、下拉选择组件配置
      if (
        module.type === this.module.radio.type ||
        module.type === this.module.checkbox.type ||
        module.type === this.module.select.type
      ) {
        this.moduleConfig.editList = module.itemList.join("##");
      }

      // 数字输入框组件配置
      if (module.type === this.module.inputNumber.type) {
        this.moduleConfig.min = module.min;
        this.moduleConfig.max = module.max;
      }

      // 修改配置的时候默认值需要的是字符串或者数组，但是checkbox组件需要的是数组
      // 所以在里专门添加一个属性来存放，转为字符串的checkbox值
      if (module.type === this.module.checkbox.type) {
        this.moduleConfig.configValue = module.value.join("##");
      }

      // 计算公式当前显示用计算公式默认值设置
      if (module.type === this.module.count.type) {
        if (
          this.moduleConfig.nowFormulas &&
          this.moduleConfig.nowFormulas.match("=")
        ) {
          this.moduleConfig.nowFormulas =
            module.label + " = " + this.moduleConfig.nowFormulas.split("=")[1];
        } else {
          this.moduleConfig.nowFormulas = module.label + " = ";
        }
      }

      // 暂存当前修改配置中的组件
      this.targetModule = params;

      // 显示组件配置对话框
      this.modulesSetingModalToogle = true;
    },

    // 保存组件配置
    saveModuleConfig: function() {
      let params = this.targetModule;
      let row = params.rowIndex;
      let col = params.colIndex;
      let m = params.mIndex;
      let module = this.modulesData[row][col][m];

      // 组件特有配置处理
      // 单选框组件、多选框组件、下拉选择组件配置
      if (
        module.type === this.module.radio.type ||
        module.type === this.module.checkbox.type ||
        module.type === this.module.select.type
      ) {
        this.moduleConfig.itemList = this.moduleConfig.editList.split("##");
      }

      // 多选框组件配置
      if (module.type === this.module.checkbox.type) {
        let newVal = this.moduleConfig.configValue.split("##");
        // 待处理：值为空时，以##为分割符会得到一个空的值，这个空的值会影响必填项判断
        // 解决办法：1、增加判断；2、修改分割办法；
        this.moduleConfig.value = this.moduleConfig.configValue.split("##");
      }

      // 计算公式组件配置
      if (module.type === this.module.count.type) {
        // 保存有效的计算公式，内部是label和计算符号
        this.moduleConfig.validFormulas = this.moduleConfig.nowFormulas.split(
          "="
        )[1];
        // 保存计算用计算公式，内部是id和计算符号
        this.moduleConfig.countFormulas = this.moduleConfig.tempCountFormulas;
      }

      // 配置验证正确，关闭对话框
      this.$refs.configForm.validate(result => {
        if (result) {
          for (const key in this.moduleConfig) {
            if (this.moduleConfig.hasOwnProperty(key)) {
              if (
                key === "require" ||
                key === "tips" ||
                this.modulesData[row][col][m][key] !== undefined
              ) {
                // 更新表单组件数据
                // if (typeof this.modulesData[row][col][m][key] === "object") {
                //   this.modulesData[row][col][m][key] = this.deepClone(this.modulesData[row][col][m][key]);
                //   console.log(key)
                //   console.log(this.modulesData[row][col][m][key])
                // } else {
                //   this.modulesData[row][col][m][key] = this.moduleConfig[key];
                // }
                this.modulesData[row][col][m][key] = this.moduleConfig[key];
                // 更新可计算对象
                if (this.calculatorList[this.moduleConfig.id]) {
                  this.calculatorList[this.moduleConfig.id][
                    key
                  ] = this.moduleConfig[key];
                }
              }
            }
          }

          // for (let index = 0; index < this.calculator.length; index++) {
          //   if (this.moduleConfig.id === this.calculator[index].id ) {
          //     for (const key in this.moduleConfig) {
          //       if (key === "require" || key === "tips" || this.moduleConfig.hasOwnProperty(key)) {
          //         this.calculator[index][key] = this.moduleConfig[key];
          //         // 数组处理
          //         // if (Array.isArray(this.moduleConfig[key])) {
          //           // this.calculator[index][key] = [].count(this.moduleConfig[key]);
          //         // 对象处理
          //         // } else if(typeof this.moduleConfig[key] === "object") {
          //           // this.calculator[index][key] = Object.assign({}, this.moduleConfig[key]);
          //         // } else {
          //           // this.calculator[index][key] = this.moduleConfig[key];
          //         // }
          //       }
          //     }
          //   }
          // }

          // 自定义事件: 表单组件设置事件，以对象形式传递新添加的表单组件
          this.$emit("setModuels", this.modulesData[row][col][m]);

          // 退出组件配置对话框
          this.cancelConfigModal();
        }
      });
    },

    // 退出组件配置对话框
    cancelConfigModal: function() {
      // 关闭计算公式配置项
      this.formulasSettingToogle = false;
      // 关闭对话框
      this.modulesSetingModalToogle = false;

      // 清空验证信息
      setTimeout(() => {
        // 跳过错误
        this.$refs.configForm.resetFields();
      });
    },

    // 删除组件
    delModule: function(params) {
      if (this.beforDeleteModule(params.module.id)) {
      let row = params.rowIndex;
      let col = params.colIndex;
      let m = params.mIndex;

      // 触发删除表单组件事件，以对象形式传递新添加的表单组件
      if (this.nowMode === this.modeOption.detailed) {
        // 自定义事件: 删除表单组件
        this.$emit("delModuels", this.modulesData[row][col][m]);
      }

      // 删除指定组件
      this.modulesData[row][col].splice(m, 1);
      // 减少该组件的数量计数器
      if (this.modulesLength[params.module.type] > -1) {
        this.modulesLength[params.module.type] =
          this.modulesLength[params.module.type] - 1;
      }

      // 重置表单组件唯一下标计数器
      if (this.modulesLength[params.module.type] < 0) {
        this.modulesIndex[params.module.type] = -1;
      }

      // 删除可计算对象
      if (this.calculatorList[params.module.id]) {
        this.$delete(this.calculatorList, params.module.id);

        // 自定义事件：删除可计算对象
        this.$emit("delCalculator", params.module.id);
      }
      }
    },

    // 增加表单行
    rowPlus: function() {
      this.rowNumber = this.rowNumber + 1;
    },

    // 减少表单行
    rowMinus: function() {
      if (this.rowNumber > 1) {
        this.rowNumber = this.rowNumber - 1;
      }
    },

    // 增加表单行列数
    gridColPlus: function(colIndex) {
      if (this.modulesData[colIndex].length < 8) {
        this.modulesData[colIndex].push([]);
      }
    },

    // 删除表单行列数
    gridColMinus: function(colIndex) {
      let col = this.modulesData[colIndex];
      let collength = this.modulesData[colIndex].length;

      // 保留一行不删完
      if (collength > 1) {
        this.modulesData[colIndex].splice(-1, 1);
      }
    },

    // 清空当前构建的表单
    clearBuilderData: function() {
      // 表单组件数据
      this.modulesData = [[[]]];
      // 表单组件设置
      this.moduleConfig = {};
      // 表单组件长度计数器
      this.modulesLength = {};
      // 表单组件下标计数器
      this.modulesIndex = {};
      // 可计算对象
      // this.calculatorList = {};
      // 表单组件-明细 的渲染数据
      this.detailedRenderData = {};
    },

    // 获取表单结构
    getBuilderData: function() {
      // 返回当前的表单结构数据
      return this.modulesData;
    },

    // 设置列大小
    setColSize: function(params) {
      // 如果设备物理像素宽度小于768则判定当前设备为手机，返回24，让表单列占满一行
      // 前提是必须在html文件的<head>内添加
      // <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
      let deviceWidth = window.screen.width;
      if (deviceWidth && deviceWidth < 992) {
        return 24;
      }

      let col = this.modulesData[params.rowIndex];
      let collength = this.modulesData[params.rowIndex].length;
      let size = 24 / collength;

      if (size.toString().split(".").length > 1) {
        if (params.nowIndex === collength - 1) {
          return 24 / (collength + 1) * 2;
        } else {
          return 24 / (collength + 1);
        }
      } else {
        return 24 / collength;
      }
    },

    // 过滤空格
    filterBlank: function(params) {
      params.name = params.name.replace(
        /(^[\d\s\n\t]{1,}|[^\d\s\n\ta-zA-Z]{1,})/,
        ""
      );
      // this.moduleConfig = Object.assign({}, this.moduleConfig,
      //  {name:this.moduleConfig.name.replace(/(^[\d]{1,}|[^\da-zA-Z]{1,})/,"")});
    },

    // 过滤数学公式符号
    filtersLabel: function(params) {
      params.label = params.label.replace(/([\+\*\/\-\×\÷\(\)\.]+)/g, "");
    }
  },

  // 生命周期钩子： 创建组件
  created: function() {
    // if (
    //   this.nowMode === this.modeOption.detailed &&
    //   Array.isArray(this.data) &&
    //   this.data.length > 0 &&
    //   this.data[0][0]
    // ) {
    //   this.modulesData = this.data;
    // }

    // 表单构建器，只有在渲染模式下才会根据 data 属性渲染表单
    // if (this.nowMode === this.modeOption.render &&  Array.isArray(this.data) &&
    // this.data.length > 0 && this.data[0][0]) {
    //   this.modulesData = this.data;
    // }
  },

  // 生命周期钩子：组件加载完毕
  mounted: function() {
    // 防止浏览器，拖动组件，松开鼠标后，会在新窗口打开被拖动的组件或传递的数据，的默认行为
    document.body.ondrop = function(param) {
      param.preventDefault();
      param.stopPropagation();
    };
  },

  // 监视数据变动
  watch: {
    // 渲染表单行
    rowNumber: function(newValue, oldValue) {
      if (newValue > oldValue) {
        let number = newValue - oldValue;
        for (let index = 0; index < number; index++) {
          this.modulesData.push([[]]);
        }
      } else if (newValue < oldValue) {
        let number = oldValue - newValue;
        this.modulesData.splice(-1, number);
      }
    },

    // 更新渲染数据
    renderData: function(newValue, oldValue) {
      this.modulesData = newValue;
    }
  }
};
</script>
