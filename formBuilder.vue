/*
 * @Author: chenzicong
 * @Date: 2018-04-03 16:49:12
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-05-31 09:06:02
 */

<template>
  <Row style="height: 2000px;">
    <!-- 构建区域 -->
    <Col span="16">
      <!-- 表单基础信息 -->
      <Col class="form-row" span="24">
        <Row>
          <!-- 表单类型编号 -->
          <Col class="form-number-wrap" :xs="24" :sm="24" :md="16" :lg="16">
            <p>单号：&nbsp;&nbsp;{{formData.number}}</p>
          </Col>

          <!-- 表单类型名称 -->
          <Col class="form-type-wrap" :xs="24" :sm="24" :md="24" :lg="24">
            <h1>{{baseData.formTypeName}}</h1>
          </Col>

          <Form ref="baseDataForm" :model="baseData"  :label-width="100" label-position="left">
          <!-- 标题 -->
          <Col class="form-title-wrap" :xs="24" :sm="24" :md="16" :lg="16">
            <FormItem label="标题：" prop="title" required>
              <Input v-model="formData.title" disabled placeholder="必填，请对本次申请进行说明"></Input>
            </FormItem>
          </Col>

          <!-- 紧急程度 -->
          <Col class="form-status-wrap" :xs="24" :sm="24" :md="8" :lg="8">
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
      <Col class="form-row" span="24">
        <Row>
          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请人：{{formData.userName}}</p>
          </Col>

          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请部门：{{formData.department}}</p>
          </Col>

          <Col class="form-col" :xs="24" :sm="8" :md="8" :lg="8">
            <p>申请日期：{{formData.date}}</p>
          </Col>
        </Row>
      </Col>

      <!-- 表单构造区域 -->
      <Col class="form-row" span="24">
          <form-grid @save="getFormData" ref="formBuilder" :rowNumber="rowNumber"></form-grid>
      </Col>
    </Col>

    <!-- 表单项组件 -->
    <Col span="8">
      <iview-form-builder-modules @save="getFormData({action:true})" @clear="clearFormData">
      </iview-form-builder-modules>
    </Col>

    <!-- 增减表单行数 -->
    <Col class="form-row-set" span="16">
      <Row>
        <Col span="8" offset="8">
          <Row :gutter="10">
            <Col span="12">
              <Button long @click.native="rowMinus" title="删除">
                <Icon type="minus"></Icon>
              </Button>
            </Col>
            <Col span="12">
              <Button long type="primary" @click.native="rowPlus" title="增加">
                <Icon type="plus"></Icon>
              </Button>
            </Col>
          </Row>
        </Col>
      </Row>
    </Col>
  </Row>
</template>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "./styles/baseStyles";
@import "./styles/commonStyles";


// 表单编号
.form-number-wrap {
  padding: $colPadding;
  padding-bottom: 0;
}
.form-number {
  text-align: left;
}

// 表单类型
.form-type-wrap {
  height: 100px;
  padding: $colPadding;
  line-height: 65px;
  text-align: center;
  font-size: 16px;
  > h1 {
    font-weight: normal;
  }
}

// 标题、紧急程度
.form-status-wrap,
.form-title-wrap {
  padding: $colPadding;
  .ivu-form-item {
    margin-bottom: 0;
  }
}

// 表单行
.form-row {
  border: 1px solid #ccc;
  border-bottom: none;
  &:last-child {
    border: 1px solid #ccc;
  }
   // 表单列
  .form-col {
    padding: $colPadding;
    border-right: 1px solid #ccc;
    &:last-child {
      border-right: none;
    }
  }
@media screen and (max-width: 982px) {
  // 表单列
  .form-col {
    padding: $colPadding;
    border: 0;
    // border-right: 1px solid #ccc;
    // &:last-child {
    //   border-right: none;
    // }
  }
}

}

// 表单单元格行数增加
.form-row-set {
  margin-top: 10px;
}
</style>

<script>
  // 表单栅格系统
  import formGrid from './formGrid';
  // 表单组件
  import iviewFormBuilderModules from './componentes/iviewFormBuilderModules';

  export default {
    // 本组件名
    name: "form-builder",

    // 应用组件
    components: {
      formGrid,
      iviewFormBuilderModules
    },

    // 自定义属性
    props: {
      // 表单类型
      baseData: {
        default:function () {
          return {
            number: "自动获取",
              formTypeName:'',
              formType:''
          }
        }
      }
    },

    // 数据
    data() {
      return {
        // 占位用表单数据
        formData:{
          number:"自动获取",
          status:"0",
          userName:"自动获取",
          department:"自动获取",
          date:"自动获取",
        },

        // 设置表单行数
        rowSet: false,

        // 表单行数
        rowNumber: 1,
      }
    },

    // 方法
    methods: {
      // 清空表单
      clearFormData() {
        this.$refs.formBuilder.clearBuilder();
      },

      // 增加行数
      rowPlus() {
        this.rowNumber = this.rowNumber + 1;
      },

      // 减少表单行数
      rowMinus() {
        if (this.rowNumber > 1) {
          this.rowNumber = this.rowNumber - 1;
        }
      },

      // 获取构建好的表单数据
      getFormData(params){
        if (params.action) {
          this.$refs.formBuilder.saveBuilder();
        } else if(params.data) {
          this.$emit("save", this.baseData, params.data);
        }
      }
    }
  }
</script>
