/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:50:02
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 10:37:20
 */


<template>
  <Row class="iview-form-builder-sidebar-wrap">
    <!-- 菜单导航 -->
    <Col span="24">
      <Menu mode="horizontal" theme="light" active-name="single" @on-select="toogleModulesGroup">
          <MenuItem name="single">
              <Icon type="ios-paper"></Icon>
              控件库
          </MenuItem>
          <MenuItem name="suite">
              <Icon type="settings"></Icon>
              套件库
          </MenuItem>
      </Menu>
    </Col>

    <!-- 表单元素&操作按钮 -->
    <Col span="24">
      <!-- 表单构建器：侧边栏 -->
      <div class="iview-form-builder-sidebar">
        <!-- 表单构建器：表单元素 -->
        <!-- 控件库 -->
        <template v-if="singleShow">
          <div class="builder-form-element" v-for="(element, index) in nowModules.modules" :key="element + index"
          draggable="true" @dragstart="drag({type: element.type}, $event)">
            <div class="builder-form-element-content">
              <Icon type="ios-browsers-outline" size="14"></Icon>&nbsp;
              {{element.title}}
            </div>
          </div>
        </template>

        <!-- 套件库 -->
        <template v-if="suiteShow">
          <div class="builder-form-element" v-for="(element, index) in nowModules.groups" :key="element + index"
          draggable="true" @dragstart="drag({type: element.type}, $event)">
            <div class="builder-form-element-content">
              <Icon type="ios-browsers-outline" size="14"></Icon>&nbsp;
              {{element.title}}
            </div>
          </div>
        </template>
      </div>
    </Col>

    <!-- <Col span="24"> -->
        <!-- 表单构建器：构建操作 -->
        <!-- <div class="iview-form-builder-sidebar-active"> -->
          <!-- 清空构建器内容 -->
          <!-- <div class="active-item">
            <Button type="ghost" size="large" long @click="clearEvent" title="清空">清空</Button>
          </div> -->

          <!-- 保存构建器数据 -->
          <!-- <div class="active-item">
            <Button type="primary" size="large" long @click="saveEvent" title="保存">保存</Button>
          </div>
        </div> -->
    <!-- </Col> -->
  </Row>
</template>


<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/sidebar";
</style>


<script>
// =============== 导入模块 ===============
// 配置文件
import config from "../config";

export default {
  // 组件名
  name: "iview-form-builder-sidebar",

  // 组件属性
  props: {
    mode: {
      default: config.mode.builder
    },
    // 设备
    device: {
      default: config.devices.desktop
    },

    // 元素列表
    modules: {
      default: function () {
        return {
          modules: config.formElement,
          groups: config.formElementGroups
        }
      }
    }
  },

  // 数据
  data() {
    return {
      // 配置信息
      config: config,
      // 默认活动设备
      activerDevice: "desktop",
      // 控件库
      singleShow: true,
      // 套件库
      suiteShow: false
    };
  },

  // 计算属性
  computed: {
    // 侧边栏模式
    nowMode: function() {
      return this.mode
    },
    // 设备平台
    nowDevice: function () {
      return this.device
    },
    // 表单元素列表
    nowModules: function() {
      return this.modules
    }
  },

  // 方法
  methods: {
    // 切换组件库
    toogleModulesGroup(params){
      switch (params) {
        // 显示控件库
        case 'single':
            this.singleShow = true;
            this.suiteShow = false;
          break;

        // 显示套件库
        case 'suite':
            this.singleShow = false;
            this.suiteShow = true;
          break;
      }
    },

    // 设置拖动组件时传递的数据
    drag: function(params, $event) {
      $event.dataTransfer.setData("Text", params.type);
    },

    // 保存构建的表单数据事件
    saveEvent: function() {
      // 自定义事件：保存构建的表单数据
      this.$emit("save");
    },

    // 清空构建器数据事件
    clearEvent: function() {
      // 自定义事件：清空构建器数据事件
      this.$emit("clear");
    }
  }
};
</script>
