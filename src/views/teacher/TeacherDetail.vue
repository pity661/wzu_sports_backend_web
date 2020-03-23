<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/teachers' }">教师管理</el-breadcrumb-item>
            <el-breadcrumb-item>教师详情</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="main-panel">
            <div class="detail">
                <el-col :span="8" class="panel">
                    <el-col :span="24" class="title">
                        基本信息
                    </el-col>
                    <div class="info">
                        <!--<img :src="this.teacherAvatar" />-->
                        <span class="avatar"></span>
                        <span>姓名：{{info.name}}</span>
                        <span>性别：{{info.isMan ? "男":"女"}}</span>
                        <span>工号：{{info.jobNo}}</span>
                        <span>年龄：{{info.age}}</span>
                    </div>
                </el-col>
                <el-col :span="15" class="panel" :offset="1">
                    <el-col :span="24" class="title">
                        教师课表
                    </el-col>
                    <div>
                        <table class="schedule">
                            <tr>
                                <td>--</td>
                                <td v-for="title in schedule.th">
                                    {{title}}
                                </td>
                            </tr>
                            <tr>
                                <td rowspan="2">上午</td>
                                <td v-for="course in schedule.morning[0]">{{course}}</td>
                            </tr>
                            <tr>
                                <td v-for="course in schedule.morning[1]">{{course}}</td>
                            </tr>
                            <tr>
                                <td>午休</td>
                                <td></td>
                                <td></td>
                                <td></td>
                                <td></td>
                                <td></td>
                            </tr>
                            <tr>
                                <td rowspan="2">下午</td>
                                <td v-for="course in schedule.afternonne[0]">{{course}}</td>
                            </tr>
                            <tr>
                                <td v-for="course in schedule.afternonne[1]">{{course}}</td>
                            </tr>
                        </table>
                    </div>
                </el-col>
            </div>

            <el-col :span="24" class="classes-panel">
                <el-col :span="24" class="title">
                    授课班级
                </el-col>

                <div v-for="item in info.classes" class="card" @click="goClass(item)"  v-loading="loading" element-loading-text="玩命加载中">
                    <div class="classes-name">
                        {{item.name}}
                    </div>
                    <div class="student-number">
                        学生总数：{{item.studentsCount}}
                    </div>
                </div>
            </el-col>

            <!--<el-col :span="24" class="page">
                <el-pagination @current-change="handleCurrentChange" :current-page.sync="currentPage" :page-size="3" layout="prev, pager, next, jumper"
                    :total="10">
                </el-pagination>
            </el-col>-->
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'

    const detailQuery = `
    query($id: Long){
        teacher(id: $id){
            name
            jobNo
            isMan
            classes{
                id
                name
                studentsCount
            }
        }
    }`;
    export default {
        data() {
            return {
                id: this.$route.params.id,
                info: {
                    name: '林金鸿',
                    sex: '男',
                    jobNo: '123',
                    isMan: '24',
                    classes: [
                        {
                            name: '营销1班',
                            id: 1,
                            studentsNum: 63
                        },
                        {
                            name: '营销2班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销3班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销2班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销3班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销2班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销3班',
                            id: 1,
                            studentsNum: 83
                        },
                        {
                            name: '营销2班',
                            id: 1,
                            studentsNum: 83
                        }
                    ]
                },
                schedule: {
                    th: ['周一', '周二', '周三', '周四', '周五'],
                    morning: [['测试', '营销121班', '', '营销3班', ''], ['', '', '营销4班', '', '']],
                    afternonne: [['测试', '营销121班', '', '营销3班', ''], ['', '', '营销5班', '', '']]
                },
                teacherAvatar: '/src/assets/user.png',
                filters: {
                    college: '',
                    major: '',
                    grade: ''
                },
                total: 0,
                loading: true
            }
        },
        methods: {
            getDetail() {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${detailQuery}`,
                    variables: {
                        "id": this.id
                    }
                })
                .then(res => {
                    _this.loading = false;
                    _this.info = res.data.data.teacher
                });
            },
            batchAddStudents() {
                this.$router.push({ path: '/addstudent' });
            },
            handleCurrentChange(val) {
                console.log(`当前页: ${val}`);
            },
            goCourses() {
                this.$router.push({ path: '/courses' });
            },
            goClass(item) {
                this.$router.push({ path: '/classdetail/' + item.id });
            }
        },
        mounted: function () {
            this.getDetail();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        min-width: 700px;
        .main-panel {
            overflow: hidden;
            margin-top: 10px;
            padding: 0 10px 0px 15px;
        }
        .panel {
            border: 1px solid #d4d4d4;
            padding: 5px 15px 15px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .info {
            text-align: center;
            span {
                display: block;
                line-height: 2;
            }
            .avatar {
                display: inline-block;
                width: 100px;
                height: 100px;
                border-radius: 50%;
                background: #ccc;
            }
        }
        .schedule {
            width: 100%;
            border-collapse: collapse;
            text-align: center;
            td {
                border: 1px solid #ebebeb;
                height: 33px;
            }
        }
        .filter-sex {
            width: 90px;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
        .classes-panel {
            overflow: hidden;
            font-size: 14px;
            .card {
                display: inline-block;
                margin: 0 10px 10px 0;
                text-align: center;
                cursor: pointer;
            }
            .classes-name {
                width: 274px;
                height: 165px;
                line-height: 165px;
                font-size: 23px;
                font-weight: bold;
                background-color: #f2f2f2;
                border: 1px solid #d4d4d4;
                border-bottom-color: transparent;
            }
            .student-number {
                width: 274px;
                height: 85px;
                line-height: 85px;
                font-size: 16px;
                background-color: #fff;
                color: #101010;
                border: 1px solid #d4d4d4;
                border-top-color: transparent;
            }
        }
    }
</style>