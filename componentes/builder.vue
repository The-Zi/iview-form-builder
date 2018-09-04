/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:12
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 16:28:06
 */


<template>
  <Row>
    <!-- 表单侧边栏 -->
    <Col span="6">
      <sidebar mode="builder" device="desktop" @save="saveEvent({action:true})" @clear="clearEvent"></sidebar>
    </Col>

    <!-- 表单区域 -->
    <!-- <Col span="8"> -->
      <!-- <Row> -->
        <Col :span="nowDevice === config.devices.mobile? 18:24">
        <!-- <Col :span="nowDevice === config.devices.mobile? 8:24" :offset="nowDevice === config.devices.mobile? 2: 0"> -->
          <!-- 移动设备背景图 -->
          <img v-if="nowDevice === config.devices.mobile" class="iview-form-builder-bg"
          src="../images/Mobile_iphone.png">

          <!-- 表单构建器栅格系统 -->
          <grid :class="{'iview-form-builder-grid-mobile': nowDevice === config.devices.mobile}"
          ref="grid" :mode="nowMode" :device="nowDevice" :renderData="nowRenderData"
          :beforDeleteModule="nowBeforDeleteModule">
          </grid>
        </Col>
      <!-- </Row> -->
    <!-- </Col> -->
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
// 栅格系统
import grid from "./grid";
// 侧边栏
import sidebar from "./sidebar";

export default {
  // 本组件名
  name: "iview-form-builder",

  // 应用组件
  components: {
    grid,
    sidebar
  },

  // 自定义属性
  props: {
    // 构建器模式
    mode: {
      default: config.mode.builder
    },
    // 显示设备
    device: {
      default: config.devices.desktop
    },
    // 渲染数据
    renderData: {
      default: Array
    },
    // 在删除组件前执行的函数
    beforDeleteModule: ""
  },

  // 计算属性
  computed: {
    nowMode: function() {
      return this.mode;
    },
    nowDevice: function() {
      return this.device;
    },
    nowRenderData: function() {
      return this.renderData;
    },
    nowBeforDeleteModule: function () {
      return this.beforDeleteModule
    }
  },

  // 数据
  data() {
    return {
      // 配置选项
      config: config
    };
  },

  // 方法
  methods: {
    // 清除事件
    clearEvent() {
      // 清空表单构建器数据
      this.$refs.grid.clearBuilderData();
    },

    // 保存事件
    saveEvent(params) {
      // 自定义事件：保存表单构建器数据
      this.$emit("saveEvent", { form: this.$refs.grid.getBuilderData() });
    },

    // 保存
    save(){
      return this.$refs.grid.getBuilderData();
    }
  }
};
</script>
