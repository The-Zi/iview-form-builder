/*
 * @Author: The-Zi
 * @Date: 2018-06-22 14:15:33
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-08-27 16:36:08
 */


<template>
<!-- 渲染表单组件 -->
<div>
    <div v-for="(modules, mIndex) in nowCol" :key="'form-modules-' + mIndex">
        <!-- 设置组件的验证规则 -->
        {{ requireSetEvent(modules) }}

        <!-- 表单组件: 预算 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === moduleGroups.budget.type
        || modules.type === '预算'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <RadioGroup v-model="modules.value" @on-change="formDataSync">
              <Radio v-for="(item, index) in modules.itemList" :label="item"
               :key="modules.name + '-' + index">{{ item }}</Radio>
               <!-- :key="modules.name + '-' + index">{{item == 0? "预算内":"预算外"}}</Radio> -->
            </RadioGroup>
        </FormItem>

        <!-- 表单组件：文本输入框 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.inputText.type
        || modules.type === '文本输入框'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"
                type="ios-gear"/>

                 <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <Input v-model="modules.value" :placeholder="modules.tips" @on-blur="formDataSync"></Input>
        </FormItem>

        <!-- 表单组件：数字输入框 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.inputNumber.type
        || modules.type === '数字输入框'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 发现可计算对象事件 -->
            {{findCalculatorEvent(modules)}}

            <!-- 表单组件 -->
            <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
            :min="modules.min" :placeholder="modules.tips" @on-change="countEvent(modules)"
            @keyup.native="($event)=>{inputNumberValClear($event, modules)}">
            </Input-number>
        </FormItem>

        <!-- 表单组件: 计算公式 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.count.type"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 发现计算公式组件事件 -->
            {{findCountFormulaModuleEvent(modules)}}

            <!-- 发现可计算对象事件 -->
            {{findCalculatorEvent(modules)}}

            <!-- 表单组件 -->
            <Input-number style="width: 100%;" v-model="modules.value" :placeholder="modules.tips"
            @on-change="convertMoneyFormatEvent(modules)"
            @keyup.native="($event)=>{inputNumberValClear($event, modules)}">
            </Input-number>

            <!-- 中文大写数字金额 -->
            <p v-if="modules.showUpper">
                <!-- 结果提示 -->
                <span v-if="modules.resultTips">{{modules.resultTips}}</span>
                <!-- 单位 -->
                <span v-if="modules.unit">({{modules.unit}})</span>
                <!-- 中文大写数字金额 -->
                {{modules.cnNum}}
            </p>
        </FormItem>

        <!-- 表单组件: 金额 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.money.type"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 发现可计算对象事件 -->
            {{findCalculatorEvent(modules)}}

            <!-- 表单组件 -->
            <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
            :min="modules.min" :placeholder="modules.tips"
            @on-change="convertMoneyFormatEvent(modules)"
            @keyup.native="($event)=>{inputNumberValClear($event, modules)}">
            </Input-number>

            <!-- 中文大写数字金额 -->
            <p v-if="modules.showUpper">
                <!-- 结果提示 -->
                <span v-if="modules.resultTips">{{modules.resultTips}}</span>
                <!-- 单位 -->
                <span v-if="modules.unit">({{modules.unit}})</span>
                <!-- 中文大写数字金额 -->
                {{modules.cnNum}}
            </p>
        </FormItem>

        <!-- 表单组件: 单选框 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.radio.type
        || modules.type === '单选'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <RadioGroup v-model="modules.value" @on-change="formDataSync">
                <Radio v-for="(item, index) in modules.itemList" :label="item"
                :key="modules.name + '-' + index"></Radio>
            </RadioGroup>
        </FormItem>

        <!-- 表单组件: 多选框 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.checkbox.type
        || modules.type === '多选'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <CheckboxGroup v-model="modules.value" @on-change="formDataSync">
                <Checkbox v-for="(item, index) in modules.itemList" :key="modules.name + index"
                :label="item"></Checkbox>
            </CheckboxGroup>
        </FormItem>

        <!-- 表单组件: 下拉框 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.select.type
        || modules.type === '下拉选择'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <Select v-model="modules.value" @on-change="formDataSync">
                <Option v-for="(item, index) in modules.itemList" :value="item" :key="modules.name + index">
                    {{item}}
                </Option>
            </Select>
        </FormItem>

        <!-- 表单组件: 多行文本 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.textarea.type
        || modules.type === '多行文本'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <Input v-model="modules.value" type="textarea" :rows="4" :placeholder="modules.tips"
            @on-change="formDataSync" @on-blur="formDataSync"></Input>
        </FormItem>

        <!-- 表单组件: 说明文本 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.textRead.type
        || modules.type === '说明文本'"
        :required="modules.require" :label="modules.label">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <p>{{modules.value}}</p>
        </FormItem>

        <!-- 表单组件: 时间日期 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.dateTime.type
        || modules.type === '时间日期'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <DatePicker v-model="modules.value"
            style="width: 100%;"
            :editable="false"
            :type="dateFormat({mode: 'date',format: modules.format})"
            :format="dateFormat({mode: 'format',format: modules.format})"
            :placeholder="modules.tips" @on-change="formDataSync">
            </DatePicker>
        </FormItem>

        <!-- 表单组件: 时间日期范围 -->
        <FormItem class="iview-form-builder-modules-wrap" v-if="modules.type === module.dateTimeRange.type
        || modules.type === '时间日期范围'"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作组件 -->
            <div class="form-modules-action" v-if="nowMode !== modeOption.renderDetailed">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="settingEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex})" type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElementEvent({rowIndex: modulesData.rowIndex, colIndex: modulesData.colIndex,
                mIndex: mIndex, module: modules})"/>
            </div>

            <!-- 表单组件 -->
            <DatePicker v-model="modules.value"
            style="width: 100%;"
            :editable="false"
            :type="dateFormat({mode: 'range',format: modules.format})"
            :format="dateFormat({mode: 'format',format: modules.format})"
            :placeholder="modules.tips" @on-change="formDataSync"></DatePicker>
        </FormItem>
    </div>
</div>
</template>


<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/elements";
</style>


<script>
// =============== 导入模块 ===============
// 配置文件
import config from "../config";

export default {
  // 组件名
  name: "iview-form-builder-modules",

  // 数据
  data() {
    return {
      // 设备选项
      devicesOption: config.devices,
      // 组件模式选项
      modeOption: config.mode,
      // 表单组件配置
      module: config.formElement,
      moduleGroups: config.formElementGroups
    };
  },

  // 自定义属性
  props: {
    // 表单组件数据
    modulesData: {
      default: Object
    },
    // 表单组件类型
    mode: {
      default: String
    },
    // 表单组件所在表单行的下标
    rowIndex: {
      default: Number
    }
  },

  // 计算属性
  computed: {
    // 表单列
    nowCol: function() {
      return this.modulesData.col;
    },
    // 表单列
    nowMode: function() {
      return this.mode;
    },
    // 表单组件所在表单行的下标
    nowRowIndex: function () {
        return this.rowIndex
    }
  },

  // 方法
  methods: {
    // 表单组件删除事件
    delElementEvent: function(params) {
      // 自定义事件：删除表单组件
      this.$emit("delete", params);
    },

    // 表单组件设置事件
    settingEvent: function(params) {
      // 自定义事件：设置表单组件
      this.$emit("setting", params);
    },

    // 转换成货币格式事件
    convertMoneyFormatEvent: function(params) {
      // 自定义事件：转换成货币格式
      this.$emit("moneyFormat", params);
      // 计算事件
      this.countEvent(params);
    },

    // 清空数字输入框的值
    inputNumberValClear($event, params){
        if (!$event.target.value) {
            // 清空
            params.value = null;
            // 计算事件
            this.countEvent(params);
        }
    },

    // 过滤空格
    filterBlank: function($event, modules) {
        // if ($event.keyCode >= 48 && $event.keyCode <= 57) {
        //     console.log("number")
        // } else {
        //     console.log("NaN")
        // }
        // console.log($event)
        // console.log(this.config.keyCodes)
        let val = modules.value.replace(/(^[\0]{2,}|[^\d]{1,}|[^\d\.]{1,}|[\.]{2,})/, "");

        // 值不能超出限制范围
        if (Number(val) > Number(modules.max)) {
            modules.value = modules.max;
        } else if (Number(val) < Number(modules.min)) {
            modules.value = modules.min;
        } else {
            modules.value = val;
        }
    },

    // 计算事件
    countEvent: function(params) {
      // 自定义事件：计算
      this.$emit("count");

      // 触发计算事件的同事执行数据同步，主要用于必填项验证
      this.formDataSync();
    },

    // 必填项设置事件
    requireSetEvent: function(params) {
      //  自定义事件：必填项设置
      this.$emit("findRequire", params);
    },

    // 发现计算公式组件事件
    findCountFormulaModuleEvent: function(params) {
      // 自定义事件：发现计算公式组件
      this.$emit("findCountFormula", params);
    },

    // 发现可计算对象事件
    findCalculatorEvent: function(params) {
      // 自定义事件：发现可计算对象
      this.$emit("findCalculator", params);
    },

    // 时间日期组件格式设置
    dateFormat: function(params) {
      // 组件类型
      if (params.mode === "range" || params.mode === "date") {
        switch (params.mode) {
          // 时间日期
          case "date":
            if (params.format === 1) {
              return "date";
            } else if (params.format === 2) {
              return "datetime";
            } else {
              return "date";
            }
            break;

          // 时间日期范围
          case "range":
            if (params.format === 1) {
              return "daterange";
            } else if (params.format === 2) {
              return "datetimerange";
            } else {
              return "daterange";
            }
            break;
        }

        // 显示格式
      } else if (params.mode === "format") {
        switch (params.format) {
          case 1:
            return "yyyy年MM月dd日";
            break;

          case 2:
            return "yyyy年MM月dd日 HH:mm";
            break;
        }
        // 默认格式
        return "yyyy年MM月dd日";
      }
    },

    // 表单数据同步事件
    formDataSync() {
      //  自定义事件：数据同步
      this.$emit("syncEvent");
    }
  }
};
</script>
