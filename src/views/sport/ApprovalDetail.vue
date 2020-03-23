<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/approval' }">未达标运动审批</el-breadcrumb-item>
            <el-breadcrumb-item>未达标运动明细</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="main-panel">
            <el-table :data="tableData" style="width: 100%">
                <el-table-column prop="name" label="姓名">
                </el-table-column>
                <el-table-column prop="student_id" label="学号" width="120">
                </el-table-column>
                <el-table-column prop="project" label="运动项目" width="120">
                </el-table-column>
                <el-table-column prop="time" label="运动时长" width="120">
                </el-table-column>
                <el-table-column prop="distance" label="运动距离" width="120">
                </el-table-column>
                <el-table-column prop="speed" label="速度">
                </el-table-column>
                <el-table-column prop="step" label="步数">
                </el-table-column>
                <el-table-column prop="creat" label="运动记录创建时间" width="150">
                </el-table-column>
                <el-table-column prop="reason" label="申请理由" width="120">
                </el-table-column>
                <el-table-column prop="status" label="状态">
                </el-table-column>
            </el-table>
            <div class="reason">
                <el-col :span="24">
                    <el-col :span="24" class="title">
                        详细申请原因
                    </el-col>
                    <div class="info">
                        如果你无法简洁的表达你的想法，那只说明你还不够了解它。
                    </div>
                </el-col>
                <el-col :span="24">
                    <el-col :span="24" class="title">
                        截图
                    </el-col>
                    <div class="info">
                        <img src="xx.png" alt="">
                        <img src="xx.png" alt="">
                        <img src="xx.png" alt="">
                    </div>
                </el-col>
            </div>
            <div class="operate-panel">
                <el-button type="primary" @click="pass">通过审批</el-button>
                <el-button type="danger" @click="deny">驳回</el-button>
            </div>

            <!-- 驳回理由弹窗 -->
            <el-dialog title="驳回理由" :visible.sync="dialogTableVisible" size="tiny">
                <el-form>
                    <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="请输入内容" v-model="deny_reason">
                    </el-input>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="dialogTableVisible = false">取 消</el-button>
                    <el-button type="danger" @click="submitDeny">确认驳回</el-button>
                </div>
            </el-dialog>

        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                dialogTableVisible: false,
                deny_reason: '驳回驳回！',
                tableData: [{
                    student_id: '20170516',
                    name: '王小虎',
                    project: '快走',
                    time: '20:00:00',
                    distance: 8.42,
                    speed: 72,
                    status: '未审批',
                    step: 40,
                    creat: '2017 20:00:00',
                    reason: '产品bug',
                    operate: '查看明细'
                }]
            }
        },
        methods: {
            pass() {
                console.log(`通过审批`);
            },
            deny() {
                this.dialogTableVisible = true;
            },
            submitDeny() {
                console.log('提交deny理由');
                this.dialogTableVisible = false;
            }
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .main-panel {
            margin-top: 10px;
        }
        .table-panel {
            margin-top: 20px;
        }
        .title {
            line-height: 2.5;
            font-weight: bold;
            font-size: 14px;
        }
        .operate-panel {
            text-align: center;
        }
    }
</style>