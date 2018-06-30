/*
 * @Author: The-Zi
 * @Date: 2018-06-22 14:15:33
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-06-22 14:38:34
 */


<template>
<!-- 渲染表单元素 -->
<div>
    <div v-for="(modules, mIndex) in col" :key="'form-modules-' + mIndex">
        <!-- 表单元素: 预算 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.budget.type"
        :required="modules.require" :label="modules.label" :prop="modules.name">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <RadioGroup v-model="modules.value">
              <Radio v-for="(item, index) in modules.itemList" :label="item"
               :key="modules.name + '-' + index">{{item === 0? "预算内":"预算外"}}</Radio>
            </RadioGroup>
        </FormItem>

        <!-- 表单元素：文本输入框 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.inputText.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"
                type="ios-gear"/>

                 <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Input v-model="modules.value" :placeholder="modules.tips"></Input>
        </FormItem>

        <!-- 表单元素：数字输入框 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.inputNumber.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
            :min="modules.min" :placeholder="modules.tips">
            </Input-number>
        </FormItem>

        <!-- 表单元素: 计算公式 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.count.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Input-number style="width: 100%;" v-model="modules.value" :placeholder="modules.tips"
            @on-change="moneyFormat(modules)">
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

        <!-- 表单元素: 金额 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.money.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Input-number style="width: 100%;" v-model="modules.value" :max="modules.max"
            :min="modules.min" :placeholder="modules.tips" @on-change="moneyFormat(modules)">
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

        <!-- 表单元素: 单选框 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.radio.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <RadioGroup v-model="modules.value">
                <Radio v-for="(item, index) in modules.itemList" :label="item"
                :key="modules.name + '-' + index"></Radio>
            </RadioGroup>
        </FormItem>

        <!-- 表单元素: 多选框 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.checkbox.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <CheckboxGroup v-model="modules.value">
                <Checkbox v-for="(item, index) in modules.itemList" :key="modules.name + index"
                :label="item"></Checkbox>
            </CheckboxGroup>
        </FormItem>

        <!-- 表单元素: 下拉框 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.select.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Select v-model="modules.value">
                <Option v-for="(item, index) in modules.itemList" :value="item" :key="modules.name + index">
                    {{item}}
                </Option>
            </Select>
        </FormItem>

        <!-- 表单元素: 多行文本 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.textarea.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <Input v-model="modules.value" type="textarea" :rows="4" :placeholder="modules.tips"></Input>
        </FormItem>

        <!-- 表单元素: 说明文本 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.textRead.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <p>{{modules.value}}</p>
        </FormItem>

        <!-- 表单元素: 时间日期 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.dateTime.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <DatePicker v-model="modules.value"
            :editable="false"
            :type="dateFormat({mode: 'date',format: modules.format})"
            :format="dateFormat({mode: 'format',format: modules.format})"
            :placeholder="modules.tips">
            </DatePicker>
        </FormItem>

        <!-- 表单元素: 时间日期范围 -->
        <FormItem class="form-elements-wrap" v-if="modules.type === element.dateTimeRange.type"
        :required="modules.require" :label="modules.label">
            <!-- 操作元素 -->
            <div class="form-elements-action">
                <!-- 设置 -->
                <Icon class="action" size="16" color="#a7a7a7" title="设置"
                @click.native="setting({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex, mIndex: mIndex})"
                type="ios-gear"/>

                <!-- 删除 -->
                <Icon class="action" size="16" color="#a7a7a7" title="删除" type="android-close"
                @click.native="delElement({rowIndex: elementsData.rowIndex, colIndex: elementsData.colIndex,
                mIndex: mIndex, element: modules})"/>
            </div>

            <!-- 表单元素 -->
            <DatePicker v-model="modules.value"
            :editable="false"
            :type="dateFormat({mode: 'range',format: modules.format})"
            :format="dateFormat({mode: 'format',format: modules.format})"
            :placeholder="modules.tips"></DatePicker>
        </FormItem>
    </div>
</div>
</template>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/baseStyles";
@import "../styles/commonStyles";
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
}
</style>

<script>
// =============== 导入模块 ===============
// 配置文件
import config from '../config';

  export default {
    // 组件名
    name: "iview-form-builder-elements",

    // 数据
    data() {
      return {
        // 表单元素配置
        element: config.formElement,
      }
    },

    // 自定义属性
    props: {
        // 表单元素数据
        elementsData: {
            default: {}
        }
    },

    // 计算属性
    computed: {
        // 表单列
        col: function(){
            return this.elementsData.col
        }
    },

    // 方法
    methods: {
        // 删除表单元素
        delElement: function (params) {
            this.$emit("delete", params)
        },

        // 设置表单元素
        setting: function (params) {
            this.$emit("setting", params)
        },

        // 货币格式
        moneyFormat: function (params) {
            this.$emit("moneyFormat", params)
        },

        // 时间日期组件格式设置
        dateFormat: function (params) {
            // 组件类型
            if (params.mode === "range" || params.mode === "date") {
                switch (params.mode) {
                    case "date":
                        if (params.format === 1) {
                            return "date"
                        } else if (params.format === 2) {
                            return "datetime"
                        }
                    break;

                    case "range":
                        if (params.format === 1) {
                            return "daterange"
                        } else if (params.format === 2) {
                            return "datetimerange"
                        }
                    break;
                }
            // 显示格式
            } else if (params.mode === "format") {
                switch (params.format) {
                    case 1:
                        return "yyyy年MM月dd日"
                    break;

                    case 2:
                        return "yyyy年MM月dd日 HH:mm"
                    break;
                }
            }
      },
    }
  }
</script>
