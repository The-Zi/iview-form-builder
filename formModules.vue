/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:50:02
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-05-30 17:56:48
 */

<template>
  <Row>
    <Col span="24">
      <div :class="modulesClass">
        <div class="items" v-for="item in modulesItems" :key="item" draggable="true"
        @dragstart="drag({type: item}, $event)">
          {{item}}
        </div>
        <div class="form-active">
          <Button class="active-item" type="error" @click="clear" long>清空</Button>
          <Button class="active-item" type="success" @click="save" long>保存</Button>
        </div>
      </div>
    </Col>
  </Row>
</template>

<style lang="scss">
// =============== 导入样式文件 ===============
@import "./styles/baseStyles";
@import "./styles/commonStyles";


// 表单组件
.form-modules {
  position: fixed;
  right: 50px;
  top: 122px;
  z-index: 1000;
  border: 1px solid #ccc;
  background-color: #fff;
  width: 200px;
  padding: 5px 5px;
  .items {
    height: 30px;
    width: 100%;
    padding: 0 5px;
    margin: 5px 0;
    line-height: 30px;
    background-color: #f3f3f3;
  }

  .form-active {
    .active-item {
      margin: 5px 0;
    }
  }
}
</style>

<script>
  export default {
    // 本组件名
    name: "form-modules",

    // 数据
    data() {
      return {
        modulesClass: ["form-modules"],
        modulesItems: ["预算","文本输入框", "数字输入框", "单选", "多选", "下拉选择", "多行文本", "时间日期",
        "时间日期范围", "说明文本"]
      }
    },

    // 方法
    methods: {
      // 设置拖动传递的数据
      drag:function(params, $event){
        $event.dataTransfer.setData("Text", params.type);
      },
      // 保存表单
      save: function () {
        this.$emit("save");
      },
      // 清空表单
      clear: function () {
        this.$emit("clear");
      }
    }
  }
</script>
