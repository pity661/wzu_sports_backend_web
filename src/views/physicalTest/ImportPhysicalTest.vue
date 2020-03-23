<template>
    <div>
        <el-form :inline="true" :model="filters">
            <el-form-item label="学年">
                <el-select class="sm" v-model="filters.schoolYear" clearable placeholder="学年">
                    <el-option label="2017-2018" value="2017-2018"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="学期">
                <el-select class="sm" v-model="filters.term" clearable placeholder="学期">
                    <el-option label="第一学期" value="第一学期"></el-option>
                    <el-option label="第二学期" value="第二学期"></el-option>
                </el-select>
            </el-form-item>
        </el-form>
        <div style="width:30%;padding-left:30px">
            <el-upload
                class="upload-demo"
                drag
                :action="action"
                :data="filters"
                :before-upload="before"
                :on-success="success"
                :on-error="failed"
                limit="1"
                multiple>
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">Warning:请严格按照体测数据格式上传(只能上传.xls文件)</div>
            </el-upload>
        </div>
    </div>
    
</template>

<script>
    import resources from '../../resources'
    export default {
        data() {
            return {
                //action: resources.importSportsCourse(),
                action: resources.importPhysicalTest(),
                filters: {
                    schoolYear: '',
                    term: '',
                    universityId: sessionStorage.getItem('universityId'),
                    fileName: ''
                },
            }
        },
        methods: {
            failed() {
                this.mistake("数据格式错误,检查后重新上传")
            },
            success(res) {
                this.succeed(res)
            },
            before(file) {
                if (this.filters.schoolYear == '' || this.filters.term == '') {
                    this.lackMessage("请先选择学年、学期");
                    return false;
                }
                var xlsExt = new Array("xls");//excel文件的后缀名
                var index = file.name.lastIndexOf(".");
                var ext = file.name.substr(index+1);
                if (ext != xlsExt) {
                    this.lackMessage("请上传以'.xls'为后缀的文件");
                    return false;
                } else {
                    this.filters.fileName = file.name;
                    return true;
                }
            },
            succeed(message){
                this.$notify({
                    title: '成功',
                    message: message,
                    type: 'success'
                });
            },
            lackMessage(mistakeMes){
                this.$notify({
                    title: '警告',
                    message: mistakeMes,
                    type: 'warning'
                });
            },
            mistake(mistakeMes){
                this.$notify.error({
                    title: '错误',
                    message: mistakeMes,
                });
            }
        }
    }
</script>

<style lang="scss" scoped>
    .avatar-uploader .el-upload {
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #20a0ff;
    }
    .avatar-uploader-icon {
        border: 1px dashed #d9d9d9;
        font-size: 28px;
        color: #8c939d;
        width: 200px;
        height: 178px;
        line-height: 178px;
        text-align: center;
    }
    .avatar {
        width: 200px;
        height: 200px;
        display: block;
    }
</style>






