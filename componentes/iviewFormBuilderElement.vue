/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:50:02
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-05-30 17:56:48
 */

<template>
  <Row>
    <Col span="24">
      <!-- 表单构建器：表单元素 -->
      <div class="iview-form-builder-element">
        <!-- 表单构建器：element显示设备切换按钮 -->
        <Button v-for="(item, index) in deviceList" :key="'device' + index" :type="item.type"
        shape="circle" :title="item.desc" @click.native="deviceToggle" long>
            <Icon :type="item.icon.type" :size="item.icon.size"></Icon>&nbsp;&nbsp;{{item.title}}
        </Button>


        <!-- 表单构建器：表单元素列表 -->
        <div class="form-element" v-for="(item, index) in elementList"
        :key="item + index" draggable="true" @dragstart="drag({type: item.type}, $event)">
          {{item.title}}
        </div>

        <!-- 表单构建器：构建操作 -->
        <div class="builder-active">
          <Button class="active-item" type="ghost" shape="circle" @click="clear" title="重置">
            <Icon type="ios-close-empty" size="25"></Icon>
          </Button>

          <Button class="active-item" type="ghost" shape="circle" @click="save" title="保存">
            <Icon type="ios-checkmark-empty" size="25"></Icon>
          </Button>
        </div>
      </div>
    </Col>
  </Row>
</template>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/baseStyles";
@import "../styles/commonStyles";


// 表单构建器：表单组件
.iview-form-builder-element {
  position: fixed;
  right: 25px;
  top: 122px;
  z-index: 1000;
  // position: absolute;
  // right: 0;
  // top: 0;
  border: 1px solid #ccc;
  background-color: #fff;
  width: 150px;
  padding: 5px 5px;

  // 表单构建器：表单元素列表
  .form-element {
    height: 30px;
    width: 100%;
    padding: 0 5px;
    margin: 5px 0;
    line-height: 30px;
    background-color: #f3f3f3;
  }

  // 表单构建器：构建操作
  .builder-active {
    .active-item {
      margin: 5px 0;
    }
  }
}
</style>

<script>
  export default {
    // 本组件名
    name: "iview-form-builder-element",

    // 组件属性
    props:{
      // 设备
      device: {
        default: Array
      },

      // 表单组件元件
      element: {
        default: Array
      }
    },

    // 数据
    data() {
      return {
        // 显示设备列表
        deviceList: [
          {
            title: "Browser",
            status: true,
            active: {
              title: "",
              btnType: "info"
            },
            type: "ghost",
            desc: "查看PC端浏览器效果",
            icon: {
              type: "android-desktop",
              size: 14,
            },
          },
          {
            title: "Phone",
            type: "ghost",
            desc: "查看移动端效果",
            icon: {
              type: "iphone",
              size: 14,
            },
          }
        ],
        // 显示设备切换
        deviceBtnType: {
          default: "ghost",
          active: "info",
          browser: "info",
          phone: "ghost",
        },
        // 表单构建器：表单元素列表
        elementList: [
          {
            title: "预算",
            type: "budget",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "文本输入框",
            type: "inputText",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "数字输入框",
            type: "inputNumber",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "单选",
            type: "radio",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "多选",
            type: "checkbox",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "下拉选择",
            type: "select",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "多行文本",
            type: "textarea",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "说明文本",
            type: "textRead",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "时间日期",
            type: "dateTime",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
          {
            title: "时间日期范围",
            type: "dateTimeRange",
            icon: {
              name: "",
              size: "",
              color: "",
            },
          },
        ],
        // modulesItems: ["预算","文本输入框", "数字输入框", "单选", "多选", "下拉选择", "多行文本", "时间日期",
        // "时间日期范围", "说明文本"],
      }
    },

    // 方法
    methods: {
      // 切换显示设备
      deviceToggle:function () {

      },
      // 设置拖动组件时传递的数据
      drag:function(params, $event){
        $event.dataTransfer.setData("Text", params.type);
      },
      // 保存构建的表单数据
      save: function () {
        this.$emit("save");
      },
      // 清空构建器
      clear: function () {
        this.$emit("clear");
      },
    },
  }
</script>
