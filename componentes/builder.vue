/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:12
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 16:28:06
 */


<template>
  <Row>
    <!-- <Col span="24">
      <Row>
        <Col span="8" offset="8">
          <img class="iview-form-builder-bg" src="../images/Mobile_iphone.png">

        </Col>
      </Row>
    </Col> -->
    <!-- 表单区域 -->
    <Col span="24">
      <Row>
        <Col span="8" offset="8">
          <Row>
            <!-- 表单基础信息 -->
            <Col class="iview-form-builder-row" span="24">
              <Row>
                <!-- 表单类型编号 -->
                <Col class="form-number-wrap" span="24">
                  <p>单号：&nbsp;&nbsp;{{formData.number}}</p>
                </Col>

                <!-- 表单类型名称 -->
                <Col class="form-type-wrap" span="24">
                  <h1>{{baseData.formTypeName}}</h1>
                </Col>

                <Form ref="baseDataForm" :model="baseData"  :label-width="100" label-position="left">
                <!-- 标题 -->
                <Col class="form-title-wrap" span="24">
                  <FormItem label="标题：" prop="title" required>
                    <Input v-model="formData.title" disabled placeholder="必填，请对本次申请进行说明"></Input>
                  </FormItem>
                </Col>

                <!-- 紧急程度 -->
                <Col class="form-status-wrap" span="24">
                  <FormItem label="紧急程度：" prop="status" required>
                    <RadioGroup v-model="formData.status">
                      <Radio :label="0" disabled>一般</Radio>
                      <Radio :label="1" disabled>紧急</Radio>
                    </RadioGroup>
                  </FormItem>
                </Col>
                </Form>
              </Row>
            </Col>

            <!-- 表单申请信息 -->
            <Col class="iview-form-builder-row" span="24">
              <Row>
                <Col class="iview-form-builder-col" span="24">
                  <p>申请人：{{formData.userName}}</p>
                </Col>

                <Col class="iview-form-builder-col" span="24">
                  <p>申请部门：{{formData.department}}</p>
                </Col>

                <Col class="iview-form-builder-col" span="24">
                  <p>申请日期：{{formData.date}}</p>
                </Col>
              </Row>
            </Col>

            <!-- 表单元素渲染区域 -->
            <Col span="24">
              <grid :mode="nowMode" ref="grid"></grid>
            </Col>
          </Row>
        </Col>
      </Row>
    </Col>

    <!-- 表单侧边栏 -->
    <Col span="24">
      <sidebar mode="builder" device="desktop" :formElementList="formElementList" @save="saveEvent({action:true})"
      @clear="clearEvent"></sidebar>
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
    // 构建器模式
    mode: {
      default: "builder"
    }
  },

  // 计算属性
  computed: {
    nowMode: function () {
      return this.mode
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

      // 表单元素
      formElementList: config.formElement
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
      this.$emit("saveEvent", {
        base: this.baseData,
        form: this.$refs.grid.getBuilderData()
      });
    }
  }
};
</script>
