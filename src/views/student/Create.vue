<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/students' }">学生管理</el-breadcrumb-item>
            <el-breadcrumb-item>批量创建学生账号</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="main-panel">
            <span class="title">输入初始密码</span>
            <el-input class="pass-input" size="large" v-model="defaultPassword" placeholder="输入初始密码"></el-input>
            <span class="title">导入学生信息</span>
            <el-upload class="file-uploader" action="" :show-file-list="false" :on-success="uploadSuccess">
                <img v-if="fileUrl" :src="fileUrl" class="file">
                <i v-else class="el-icon-plus file-uploader-icon"></i>
            </el-upload>

            <el-button class="create-btn" type="primary" @click="batchAddStudents">创建账号</el-button>
        </div>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                defaultPassword: 123456,
                fileUrl: ''
            }
        },
        methods: {
            batchAddStudents() {
                console.log('批量账号');
            },
            uploadSuccess(res, file) {
                this.fileUrl = URL.createObjectURL(file.raw);
            }
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        .main-panel {
            padding-top: 50px;
            text-align: center;
            .title {
                margin: 10px 0 20px 0;
                display: block;
                font-size: 18px;
                color: #828282;
                line-height: 26px;
                font-weight: bold;
            }
        }
        .pass-input {
            width: 432px;
        }
        .file-uploader .el-upload {
            border: 1px dashed #d9d9d9;
            border-radius: 6px;
            cursor: pointer;
            position: relative;
            overflow: hidden;
        }
        .file-uploader .el-upload:hover {
            border-color: #20a0ff;
        }
        .file-uploader-icon {
            font-size: 28px;
            color: #8c939d;
            width: 208px;
            height: 46px;
            line-height: 46px;
            text-align: center;
            border: 1px dotted #bbb;
            margin-bottom: 20px;
        }
        .file {
            width: 178px;
            height: 178px;
            display: block;
        }
        .create-btn {
            width: 415px;
            height: 70px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 8px;
        }
    }
</style>