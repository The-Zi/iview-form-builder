/*
 * @Author: chenzicong
 * @Date: 2018-04-17 16:35:42
 * @Last Modified by: The-Zi
 * @Last Modified time: 2018-07-11 16:41:11
 */

<template>
    <Row>
        <Col span="24">
            <!-- 帮助信息 -->
            <div class="file-upload-tips">
                <p>
                    1、文件大小不能大于：
                    <span class="tips-highlight">{{this.maxSize / 1024}}M</span>
                </p>
                <p>
                    2、只允许上传后缀名为：
                    <span class="tips-highlight">{{this.acceptType.join("、")}}</span>类型的文件
                </p>
            </div>

            <!-- 待上传文件列表 -->
            <table class="file-table-wrap" border="1" cellpadding="0" cellspacing="0">
                <th style="width: 70%;">待上传列表</th>
                <th style="width: 10%;">创建人</th>
                <th style="width: 10%;">上传时间</th>
                <th style="width: 10%;">操作</th>
                <!-- 文件列表 -->
                <tr v-for="(item, index) in fileList" :key="index">
                    <!-- 备注 -->
                    <td style="text-align: center;">
                        {{item.title}}
                    </td>
                    <td style="text-align: center;">
                        {{item.userName}}
                    </td>
                    <td style="text-align: center;">
                        {{item.createTimeTemp}}
                    </td>

                    <!-- 操作 -->
                    <td>
                        <a v-if="item.url" :href="[item.url,baseUrl]|urlFilter">
                            <Icon class="action" title="下载" type="arrow-down-a" size="16"></Icon>
                        </a>

                        <Poptip @on-ok="delFile(index)" transfer confirm title="您确定要删除吗？">
                            <Icon class="action" title="删除" type="close" size="14"></Icon>
                        </Poptip>
                    </td>
                </tr>
            </table>
        </Col>

        <!-- 上传控件 -->
        <Col class="upload-row" span="24">
            <Row>
                <Col class="upload-col" span="8" :offset="8">
                    <upload ref="upload" :action="actionUrl"
                        :max-size="maxSize"
                        :show-upload-list="false"
                        :before-upload="stopDefaultUpload"
                        :on-success="success"
                        :with-credentials="true"
                        :on-error="error">
                        <Button type="primary" long icon="plus" :loading="loadingStatus">
                        添加附件
                        </Button>
                    </upload>
                </Col>
            </Row>
        </Col>

        <!-- 输入文件备注 -->
        <Modal v-model="fileRemarkModal" :closable="false" :mask-closable="false">
            <div slot="header">
                <h2>请输入文件备注</h2>
            </div>

            <!-- 备注 -->
            <Input v-model="fileRemark" @keyup.native="filtering" placeholder="附件备注信息" :maxlength="32"></Input>
            <p v-show="remakError" style="color:red;">不能为空</p>

            <div slot="footer">
                <Button type="default" @click.native="cancelFileRemak">不需要备注</Button>
                <Button type="primary" @click.native="addFileRemark">添加备注</Button>
            </div>
        </Modal>
    </Row>
</template>

<script>
// api
import { baseUrl } from '@/libs/api';
import { deleteCommFileResource } from '@/libs/file_api';

export default {
    // 组件名
    name: "file-remark-upload",

    props:{
        // 文件上传地址
        actionUrl: {
            default: ""
        },
        // 渲染数据
        renderData:{
            default: Array
        },
        // 文件上传大小限制
        fileMaxSize: {
            default: 2048
        },
        // 允许上传的文件类型
        acceptType: {
            default: ()=>{
                return ["jpg", "gif", "png", "pdf", "doc", "docx", "xls", "xlsx"]
            }
        }
    },

    // 数据
    data () {
        return {
           // 服务器地址
           baseUrl:baseUrl,
           // 待上传文件列表
           fileList: [],
           // 需要上的文件数量/下标
           fileUploadNum: [],
           // 文件上传后，返回的路径列表
           uploadResultList: [],
           // 服务器上需要删除的文件id
           delServeFileIdList:[],

           // 文件备注输入对话框
           fileRemarkModal: false,
           // 文件备注
           fileRemark:"",
           // 上传备注验证规则
           ruleValidate: {
               remark: [{required: true, message: '必填，不能为空', trigger: 'change'}]
           },
           // 结果列表
           resultList:[],
           // 删除文件结果
           delServeFileResult:[],
           // 加载状态
           loadingStatus: false,

           // 备注错误信息
           remakError: false,

           // 上传备注显示切换
           uploadRemarkToggle: true,
        }
    },

    // 计算属性
    computed:{
        // 文件最大，大小
        maxSize: function () {
            return this.fileMaxSize
        },
    },

    // 方法
    methods : {
        // 创建通用数据
        initGlobalData(){
            let date = new Date();
            let year = date.getFullYear();
            let month = date.getMonth() + 1 >= 10 ? date.getMonth() + 1 : "0" + (date.getMonth() + 1);
            let day = date.getDate() >= 10 ? date.getDate() : "0" + date.getDate();

            this.fileList[this.fileList.length - 1].userName = window.localStorage.getItem("userName");
            this.fileList[this.fileList.length - 1].createTime = date.getTime();
            this.fileList[this.fileList.length - 1].createTimeTemp = year + "-" + month + "-" + day;
        },

        // 渲染默认文件列表
        renderDefaultList(){
            let renderData = this.renderData;

            if (Array.isArray(renderData) && renderData.length > 0) {
                // 重置文件列表
                this.fileList = [];

                for (let index = 0; index < renderData.length; index++) {
                    // 格式化时间日期
                    let date = new Date(Number(renderData[index].createTime));
                    let year = date.getFullYear();
                    let month = date.getMonth() + 1 > 10 ? date.getMonth() + 1 : "0" + (date.getMonth() + 1);
                    let day = date.getDate();

                    // 添加文件
                    this.fileList.push({
                        id: renderData[index].id,
                        title: renderData[index].title,
                        file: 0,
                        userName: renderData[index].userName,
                        createTimeTemp:  renderData[index].createTime,
                        createTime:  year + "-" + month + "-" + day,
                        url: renderData[index].url,
                    });
                }
            }
        },

        // 上传成功
        success(response, file, fileList){
            this.uploadResultList.push({url:response.data});
        },

        // 上传失败
        error(error, file, fileList){
            this.fileRemark = "";
            // console.log("附件上传失败：" + error);
        },

        // 删除文件
        delFile(index){
            if (this.fileList[index].file === 0 && this.fileList[index].id) {
                this.delServeFileIdList.push(this.fileList[index].id);
                this.fileList.splice(index, 1);
            } else {
                this.fileList.splice(index, 1);
            }
        },

        // 清空待上传文件列表
        clearFileList(){
            this.fileList = [];
        },

        // 添加文件备注
        addFileRemark(){
            if (this.fileRemark !== "") {
                this.fileList[this.fileList.length - 1].title = this.fileRemark;
                this.remakError = false;
                this.fileRemarkModal = !this.fileRemarkModal;
            } else {
                this.remakError = true;
            }
        },

        // 不需要文件备注
        cancelFileRemak(){
            this.fileRemark = "";
            this.fileRemarkModal = !this.fileRemarkModal;
        },

        // 阻止默认上传行为&限制文件类型&限制文件大小
        stopDefaultUpload(file){
            // 获取选择的文件的文件类型
            let fileType = file.name.split(".");
            let nowType = fileType[fileType.length - 1];
            // 文件类型检查结果
            let checkTypeResult = false;

            // 检测文件类型是否在允许范围内
            for (let index = 0; index < this.acceptType.length; index++) {
                if (nowType.toLowerCase() === this.acceptType[index].toLowerCase()) {
                        checkTypeResult = true;
                    break;
                }
            }

            // 不允许的文件类型
            if (!checkTypeResult) {
                this.$Modal.error({
                    title: "文件类型错误",
                    content: "只允许上传后缀名为：" + this.acceptType.join("、") + " 类型的文件"
                });

            // 超出最大限制
            } else if(file.size / 1024 > this.maxSize) {
                this.$Modal.error({
                    title: "文件大小错误",
                    content: "文件大小不能超过：" + this.maxSize / 1024 + "M"
                });

            // 允许上传
            } else {
                // 重置备注信息
                this.fileRemark = "";
                // 重置备注信息错误状态
                this.remakError = false;
                // 显示输入备注信息对话框
                this.fileRemarkModal = !this.fileRemarkModal;
                // 将选择的文件添加进待上传文件列表
                this.fileList.push({file: file, title: file.name});
                // 创建通用数据
                this.initGlobalData();
            }

            return false
        },

        // 文件备注禁止输入空格
        filtering(){
            this.fileRemark = this.fileRemark.replace(/\s+/,"");
        },

        // 执行上传
        handleUpload(){
            let fileList = this.fileList;
            let delStatus = false;

            // 删除服务器端文件
            if (this.delServeFileIdList.length > 0) {
                for (let index = 0; index < this.delServeFileIdList.length; index++) {
                    deleteCommFileResource({id:this.delServeFileIdList[index]}).then(res => {
                        if (Number(res.code) === 200) {
                            // 服务器端文件删除结果
                            this.delServeFileResult.push({
                                fileListIndex:this.delServeFileIdList[index],
                                status:res.code,
                            });
                        }
                    });
                }

                // 检查删除服务端文件状态
                let checkDelServeFileStatus = setInterval(()=>{
                    if (this.delServeFileResult.length === this.delServeFileIdList.length) {
                        clearInterval(checkDelServeFileStatus);
                        delStatus = true;
                    }
                },250);
            } else {
                delStatus = true;
            }

            // 提交新文件
            if(fileList.length > 0){
                // 开启上传状态动画
                this.loadingStatus = !this.loadingStatus;

                // 提交新文件
                for (let index = 0; index < fileList.length; index++) {
                    if (fileList[index].file !== 0 && typeof fileList[index].file === "object") {
                        // 记录上传的文件下标
                        this.fileUploadNum.push({fileListIndex: index});
                        // 提交文件
                        this.$refs.upload.post(fileList[index].file);
                    }
                }

                // 检查新文件的上传情况
                if (this.fileUploadNum.length > 0) {
                    // 合并文件备注和文件url
                    let checkUploadAllStatus = setInterval(()=>{
                        if (this.uploadResultList.length > 0 && this.fileUploadNum.length > 0) {
                            if (this.uploadResultList.length === this.fileUploadNum.length && delStatus === true) {
                                clearInterval(checkUploadAllStatus);
                                for (let index = 0; index < this.fileUploadNum.length; index++) {
                                    this.resultList.push({
                                        title:this.fileList[this.fileUploadNum[index].fileListIndex].title,
                                        url: this.uploadResultList[index].url,
                                    });
                                }
                                this.loadingStatus = !this.loadingStatus;
                                this.fileList = [];
                                this.$emit("uploadEnd",this.resultList);
                            }
                        }
                    },500);
                } else {
                    // 关闭上传状态动画
                    this.loadingStatus = !this.loadingStatus;
                    this.$emit("uploadEnd",0);
                }
            } else {
                this.uploadRemarkToggle = !this.uploadRemarkToggle;
                this.$emit("uploadEnd",0);
            }
        },
    },

    // 生命周期钩子-安装完成
    mounted:function(){
        // 设置默认列表
        this.renderDefaultList();
    },

    // 监控数据变化
    watch:{
        // 渲染默认文件列表
        renderData: function (newVal, oldVal) {
            this.renderDefaultList();
        }
    },

    // 过滤器
    filters:{
        urlFilter: function (value) {
            return value[1]+"pms/file/download?filePath="+escape(value[0]);
        }
    }
}
</script>

<style lang="scss" scoped>
// =============== 导入样式文件 ===============
@import "./styles/base";
@import "./styles/common";


// 附件上传组件
.upload-row {
    margin-top: 15px;
    .ivu-upload-select {
        width: 100%!important;
    }
}

// 待上传文件列表
.file-table-wrap {
    width: 100%;
    border: 1px solid #ccc;
    > th {
        height: 30px;
        line-height: 30px;
        border: 0;
        border-right: 1px solid #ccc;
        &:last-child {
            border: 0;
        }
    }
    > tr {
        border: 0;
        &:hover {
            background-color: #fff;
        }
        > td {
            height: 30px;
            border: 0;
            border-top: 1px solid #ccc;
            border-right: 1px solid #ccc;
            text-align: center;
            &:last-child {
                border-right: 0;
            }
            .action {
                margin: 0 5px;
                cursor: pointer;
            }
        }
    }
}

// 帮助信息
.file-upload-tips {
    padding: 5px 0;
    p {
        font-size: 12px;
    }
    .tips-highlight {
        margin: 0 5px;
        color: #ff6666;
    }
}
</style>
