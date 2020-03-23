<template>
    <div class="page-container">
        <div>
            <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/version' }">版本发布管理</el-breadcrumb-item>
                <el-breadcrumb-item>版本编辑</el-breadcrumb-item>
            </el-breadcrumb>
            <el-form class="panel" ref="form" label-width="80px">
                <el-form-item label="版本名称">
                    <el-input v-model="info.versionName"></el-input>
                </el-form-item>
                <el-form-item label="版本号">
                    <el-input-number v-model="info.versionCode" :min="1" :max="100"></el-input-number>
                </el-form-item>
                <el-form-item label="上传apk" v-if="platform === 'Android'">
                    <!-- <el-input placeholder="请输入内容" v-model="info.downloadUrl">
                        <template slot="prepend">Http://</template>
                    </el-input> -->
                    <el-upload class="upload-demo" action="" :on-preview="handlePreview" :on-remove="handleRemove" :before-upload="beforeUpload">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传apk文件</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="强制升级">
                    <el-switch v-model="info.isForced" on-text="是" off-text="否"></el-switch>
                </el-form-item>
                <el-form-item label="更新日志">
                    <el-input type="textarea" :rows="20" placeholder="请输入内容" v-model="info.changeLog"></el-input>
                </el-form-item>
            </el-form>
            <el-button class="submit-btn" type="primary" @click="submit">保存</el-button>
            <el-button class="submit-btn" @click="submit">立刻发布</el-button>
        </div>
    </div>
</template>
<script>
    import resources from '../../resources'

    export default {
        data() {
            return {
                platform: this.$route.params.platform,
                // 编辑类型：'edit' 'new'
                type: this.$route.params.type,
                versionId: this.$route.query.versionId,
                info: {
                    versionName: '', // 版本名称
                    versionCode: 0, // 版本号
                    downloadUrl: '',
                    isForced: false,
                    changeLog: '',
                },
                universityId: resources.universityId,
                fileList: [
                    { name: 'food.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100' },
                ]
            }
        },
        methods: {
            submit() {
                let _this = this;
                let platformId = this.platform === 'Android' ? 0 : 1;
                let url = this.type === 'edit' ? resources.versions(this.versionId) : resources.versions();
                // 普通的ajax接口
                // 使用 application/x-www-form-urlencoded 格式化 
                // 参考：http://blog.csdn.net/fantian001/article/details/70193938
                let params = new URLSearchParams();
                params.append('versionName', this.info.versionName);
                params.append('versionCode', this.info.versionCode);
                params.append('changeLog', this.info.changeLog);
                params.append('isForced', this.info.isForced);
                params.append('platformId', platformId);
                params.append('downloadUrl', this.info.downloadUrl);
                this.$ajax.post(url, params)
                    .then(res => {
                        if (res.status < 400) {
                            _this.$router.push({ path: `/version` });
                        }
                    });
            },
            // 获取版本信息-有传id时需要获取
            getVersionInfo() {
                let _this = this;
                let url = resources.versions(this.versionId);
                // 普通的ajax接口
                // 使用 application/x-www-form-urlencoded 格式化 
                // 参考：http://blog.csdn.net/fantian001/article/details/70193938
                let params = new URLSearchParams();
                this.$ajax.get(url)
                    .then(res => {
                        this.info = res.data.obj;
                    });
            },
            handleRemove(file, fileList) {
                console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
                this.info.downloadUrl = 'www.baidu.com';
            },
            beforeUpload(file) {
                const isApk = file.type === 'apk';
                const isLt2M = file.size / 1024 / 1024 < 2;

                if (!isApk) {
                    this.$message.error('上传文件只能是 APK 格式!');
                }
                if (!isLt2M) {
                    this.$message.error('上传文件大小不能超过 2MB!');
                }
                return isApk && isLt2M;
            }
    },
    mounted: function () {
        this.getVersionInfo();
    }
}

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .panel {
            margin-top: 20px;
            width: 600px;
        }
        .submit-btn {
            margin-left: 80px;
        }
    }
</style>