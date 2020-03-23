<template>
    <div class="page-container">
        <div>
            <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
                <el-breadcrumb-item>设置异常运动指标</el-breadcrumb-item>
            </el-breadcrumb>
            <span class="title">跑步运动</span>
            <div class="panel">
                <el-form ref="form" :model="form" label-width="150px">
                    <el-form-item label="速度(m/s):">
                        <el-input v-model="form.speed"></el-input>
                    </el-form-item>
                    <el-form-item label="步幅(m):">
                        <el-input v-model="form.distancePerStep"></el-input>
                    </el-form-item>
                </el-form>
            </div>
            <el-button class="submit-btn" type="primary" @click="submit">保存</el-button>
        </div>
    </div>
</template>
<script>
    import resources from '../../resources'

    export default {
        data() {
            return {
                universityId: resources.universityId,
                form: {
                    id: '',
                    speed: '',
                    distancePerStep: '',
                }
            }
        },
        methods: {
            submit() {
                let _this = this;
                // 普通的ajax接口
                // 使用 application/x-www-form-urlencoded 格式化 
                // 参考：http://blog.csdn.net/fantian001/article/details/70193938
                let url = resources.sportDataValidateRules(this.form.id);
                let params = new URLSearchParams();
                params.append('speed', this.form.speed);
                params.append('distancePerStep', this.form.distancePerStep);

                this.$ajax.post(url, params)
                    .then(res => {
                        this.$message('设置成功~');
                        this.$router.push({ path: '/courses' });
                    });

            },
        },
        mounted: function () {
            let _this = this;
            this.$ajax.get(resources.sportDataValidateRules()).then(res => {
                _this.form = res.data.obj[0];
            })
            
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .panel {
            margin: 20px 0;
        }
        .submit-btn {
            margin-left: 80px;
        }
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .el-input {
            width: 80px;
        }
    }
</style>