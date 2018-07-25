/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:12
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 16:36:25
 */

<template>
  <Row>
    <Col span="24">
      <!-- 构建表单 -->
      <builder v-if="nowMode === config.mode.builder || nowMode === config.mode.builderMobile" ref="builder"
      :mode="mode" :baseData="nowBaseData" @saveEvent="sendFormBuilderData">
      </builder>

      <!-- 渲染表单 -->
      <render v-if="nowMode === config.mode.render" ref="render" :baseData="nowBaseData"
      :renderData="nowRenderData" @submitEvent="sendFormBuilderData"></render>

      <!-- 阅览表单 -->
      <reading v-if="nowMode === config.mode.reading" :baseData="nowBaseData" :renderData="nowRenderData"
      :keyValue="nowKeyValue"></reading>
    </Col>
  </Row>
</template>

<style lang="scss" scoped>
</style>

<script>
// =============== 导入模块 ===============
// 配置文件
import config from "./config";
// 构建表单
import builder from "./componentes/builder";
// 渲染表单
import render from "./componentes/render";
// 阅览表单
import reading from "./componentes/reading";

export default {
  // 本组件名
  name: "iview-form-builder",

  // 应用组件
  components: {
    builder,
    render,
    reading
  },

  // 自定义属性
  props: {
    // 模式
    mode: {
      default: ""
    },
    // 表单类型
    baseData: {
      default: function() {
        return {
          number: "自动获取",
          formTypeName: "",
          formType: ""
        };
      }
    },
    // 渲染数据
    renderData: {
      default: Array
    },
    // 表单键值对
    keyValue: {
      default: Object
    }
  },

  // 计算属性
  computed: {
    nowMode: function() {
      return this.mode;
    },
    nowBaseData: function() {
      return this.baseData;
    },
    nowRenderData: function() {
      return this.renderData;
    },
    nowKeyValue: function() {
      return this.keyValue;
    }
  },

  // 数据
  data() {
    return {
      // 占位用表单数据
      formData: {
        number: "自动获取",
        status: "0",
        userName: "自动获取",
        department: "自动获取",
        date: "自动获取"
      },
      // 配置信息
      config: config
    };
  },

  // 方法
  methods: {
    // 构建模式：清空表单构建器数据
    clearFormBuilderData() {
      if (this.mode === config.mode.builder) {
        this.$refs.builder.clearEvent();
      }
    },

    // 构建模式&渲染模式：发送表单构建器数据
    sendFormBuilderData(params) {
      // 自定义事件：表单构建器-构建模式，保存数据
      if (this.mode === config.mode.builder) {
        this.$emit("on-save", params);
      }

      // 自定义事件：表单构建器-渲染模式，发送当前填写的表单数据
      if (this.mode === config.mode.render) {
        this.$emit("on-submit", params);
      }
    },

    // 渲染模式：重置表单验证状态
    resetFields() {
      if (this.mode === config.mode.render) {
        this.$refs.render.resetFields();
      }
    },

    // 渲染模式：提交表单
    submit(params) {
      if (this.mode === config.mode.render) {
        this.$refs.render.handleSubmit();
      }
    }
  }
};
</script>
