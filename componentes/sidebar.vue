/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:50:02
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-18 10:37:20
 */


<template>
  <Row>
    <Col span="24">
      <!-- 表单构建器：侧边栏 -->
      <div class="iview-form-builder-sidebar">
        <!-- 表单构建器：切换显示设备 -->
        <Button class="builder-decive" v-for="(device, index) in deviceList" :key="'device' + index"
        v-if="device.status === true" :type="activerDevice === device.type ? 'info' : 'ghost'"
        @click.native="deviceToggle(device.type)" shape="circle" long :title="device.desc">
          <Icon :type="device.icon.type" :size="device.icon.size"></Icon>&nbsp;&nbsp;{{device.title}}
        </Button>


        <!-- 表单构建器：表单元素 -->
        <div class="builder-form-element">
          <Icon type="ios-browsers-outline" size="14"></Icon>&nbsp;套件
        </div>

        <div class="builder-form-element" v-for="(element, index) in elementList" :key="element + index"
        draggable="true" @dragstart="drag({type: element.type}, $event)">
          <Icon type="ios-browsers-outline" size="14"></Icon>&nbsp;
          {{element.title}}
        </div>


        <!-- 表单构建器：构建操作 -->
        <div class="builder-active">
          <!-- 清空构建器内容 -->
          <div class="active-item">
            <Button type="ghost" shape="circle" size="large" icon="close"
            @click="clearEvent" title="清空"></Button>
          </div>

          <!-- 保存构建器数据 -->
          <div class="active-item">
            <Button type="success" shape="circle" size="large" icon="checkmark"
            @click="saveEvent" title="保存"></Button>
          </div>
        </div>
      </div>
    </Col>
  </Row>
</template>


<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "../styles/sidebar";
</style>


<script>
  export default {
    // 组件名
    name: "iview-form-builder-sidebar",

    // 组件属性
    props:{
      // 设备
      device: {
        default: function () {
          return ["desktop", "mobile"]
        }
      },

      // 元素列表
      formElementList: {
        default: Object
      }
    },

    // 数据
    data() {
      return {
        // 默认活动设备
        activerDevice: 'desktop',
        // 设备列表
        deviceList: {
         desktop: {
            title: "Desktop",
            type: "desktop",
            desc: "桌面设备",
            status: false,
            icon: {
              type: "android-desktop",
              size: 14
            }
          },
          mobile: {
            title: "Mobile",
            type: "mobile",
            desc: "移动设备",
            status: false,
            icon: {
              type: "iphone",
              size: 14
            }
          }
        }
      }
    },

    // 计算属性
    computed: {
      // 表单元素列表
      elementList: function () {
        if (this.formElementList instanceof Object) {
          return this.formElementList;
        } else {
          return null
        }
      }
    },

    // 方法
    methods: {
      say(){
        // console.log(231)
      },
      // 切换显示设备
      deviceToggle:function (params) {
        this.activerDevice = params;

        // 触发设备切换事件
        this.$emit("deviceToggle", params);
      },

      // 设置使用那些设备
      setDevice(params){
        if (Array.isArray(params)) {
          if (params.length > 1) {
            // 临时对象，用于修改设备列表顺序
            let tempObject = {};

            // 选择设备
            for (let index = 0; index < params.length; index++) {
              if (this.deviceList[params[index]]) {
                // 将设备属性的第一个元素设置默认设备
                if (index === 0) {
                  this.activerDevice = this.deviceList[params[index]].type;
                  tempObject[this.activerDevice] = this.deviceList[params[index]];
                }

                // 显示指定的设备
                this.deviceList[params[index]].status = true;
              }
            }

            // 修改设备列表顺序，将传进来的设备数据第一个设为默认并提前到第一
            this.deviceList = Object.assign({},tempObject,this.deviceList);

          // 不显示设备选项
          } else {
            for (const key in this.deviceList) {
              if (this.deviceList.hasOwnProperty(key)) {
                this.deviceList[key].status = false;
              }
            }
          }
        }
      },

      // 设置拖动组件时传递的数据
      drag: function(params, $event){
        $event.dataTransfer.setData("Text", params.type);
      },

      // 保存构建的表单数据事件
      saveEvent: function () {
        // 自定义事件：保存构建的表单数据
        this.$emit("save");
      },

      // 清空构建器数据事件
      clearEvent: function () {
        // 自定义事件：清空构建器数据事件
        this.$emit("clear");
      },
    },

    // 生命周期钩子：组件安装完毕
    mounted: function () {
      // 设置默认设备
      this.setDevice(this.device);
    },

    // 监测数据变化
    watch: {
      // 实时更新设备数据
      device: function (newVal, oldVal) {
        this.setDevice(newVal);
      }
    }
  }
</script>
