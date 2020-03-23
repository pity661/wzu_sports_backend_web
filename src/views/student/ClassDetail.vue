<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/students' }">学生管理</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/students' }">班级列表</el-breadcrumb-item>
            <el-breadcrumb-item>班级详情</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="main-panel">
            <div>
                <el-col class="table-panel panel" :span="16">
                    <el-col :span="24" class="title">
                        学生
                    </el-col>
                    <el-col :span="24">
                        <el-form :inline="true" :model="filters">
                            <el-form-item>
                                <el-input v-model="filters.name" placeholder="输入学生姓名"></el-input>
                            </el-form-item>
                            <el-form-item>
                                <el-input v-model="filters.studentNo" placeholder="输入学生学号"></el-input>
                            </el-form-item>
                            <el-form-item>
                                <el-select class="filter-sex" v-model="filters.isMan" placeholder="性别">
                                    <el-option label="男" value="true"></el-option>
                                    <el-option label="女" value="false"></el-option>
                                </el-select>
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" @click="searchStudents">筛选</el-button>
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" @click="checkClassPhysical(classId)">查看该班学生体测数据</el-button>
                            </el-form-item>
                        </el-form>
                    </el-col>

                    <el-table :data="tableData" style="width: 100%" v-loading="loading" element-loading-text="玩命加载中">
                        <el-table-column label="姓名" width="180">
                            <template scope="scope">
                                <el-icon name="name"></el-icon>
                                <span class="pointer" @click="goStudentDetail(scope.row.id)">{{ scope.row.name }}</span>
                            </template>
                        </el-table-column>
                        <el-table-column prop="studentNo" label="学号" width="180">
                        </el-table-column>
                        <el-table-column prop="sex" label="性别">
                        </el-table-column>
                    </el-table>

                    <div class="page" v-if="dataCount > filters.pageSize">
                        <el-pagination @current-change="searchStudents" :current-page.sync="filters.pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                            :total="dataCount">
                        </el-pagination>
                    </div>
                </el-col>

                <el-col :span="7" :offset="1" class="panel">
                    <el-col :span="20" class="title">
                        {{info.name}}
                    </el-col>
                    <el-col :span="4" class="title">
                        {{info.studentsCount}}人
                    </el-col>
                    <el-col :span="5" :offset="7">
                        男生
                    </el-col>
                    <el-col :span="6" :offset="2">
                        女生
                    </el-col>
                    <el-col :span="5" :offset="7" class="large">
                        {{info.maleStudentsCount}}人
                    </el-col>
                    <el-col :span="6" :offset="2" class="large">
                        {{info.femaleStudentsCount}}人
                    </el-col>
                </el-col>
                <el-col :span="7" :offset="1" class="panel">
                    <el-col :span="24" class="title">
                        班级课表
                    </el-col>
                    <div class="class-teacher">
                        <span class="pointer" @click="goTeacherDetail(teacherId)">任课教师：{{teacherName}}</span>
                        <el-button @click="openSchedule">查看班级课表</el-button>
                    </div>
                </el-col>

                <el-col :span="7" :offset="1" class="panel score-panel">
                    <el-col :span="24" class="title">
                        体育成绩
                    </el-col>
                    <div class="score">
                        <span><i class="fa fa-cloud-upload"></i> <br>批量导入</span>
                        <span @click="goScore"><i class="fa fa-database"></i> <br>查看成绩</span>
                    </div>
                </el-col>
                <el-col :span="7" :offset="1" class="panel score-panel">
                    <el-col :span="24" class="title">
                        体测数据
                    </el-col>
                    <div class="score">
                        <span><i class="fa fa-cloud-upload"></i><br>批量导入</span>
                        <span @click="goData"><i class="fa fa-database"></i><br>查看体测数据</span>
                    </div>
                </el-col>

                <!-- 班级课表弹窗 -->
                <el-dialog title="班级课表" :visible.sync="dialogTableVisible">
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
                </el-dialog>
            </div>
        </div>
    </div>
</template>
<script>
    import resources from '../../resources'
    //班级详情信息
    const classQuery = `
    query($id: Long){
        class(id: $id){
            name
            studentsCount
            maleStudentsCount
            femaleStudentsCount
        }
    }`;
    // 班级学生
    const studentQuery = `
        query(
            $classId: Long
            $name: String
            $studentNo: String
            $isMan: Boolean
            $pageNumber: Int
            $pageSize: Int
        ){
            students:searchStudents(
                classId: $classId
                name: $name
                studentNo: $studentNo
                isMan: $isMan
                pageNumber: $pageNumber
                pageSize: $pageSize
            ){
                pageNum
                pageSize
                dataCount
                data{
                    id
                    name
                    studentNo
                    isMan
                }
            }
        }
    `;
    // 任课教师
    const teacherQuery = `
    query($classId: Long){
        teachers(classId: $classId){
            id
            name
        }
    }
    `;
    export default {
        data() {
            return {
                info: {
                    maleStudentsCount: 0,
                    femaleStudentsCount: 0,
                    studentsCount: 0,
                    name: '班级名称',
                },
                classId: this.$route.params.class_id,
                dialogTableVisible: false,
                schedule: {
                    th: ['周一', '周二', '周三', '周四', '周五'],
                    morning: [['营销11班', '营销121班', '', '营销3班', ''], ['', '', '营销4班', '', '']],
                    afternonne: [['营销11班', '营销121班', '', '营销3班', ''], ['', '', '营销5班', '', '']]
                },
                filters: {
                    name: '',
                    studentNo: '',
                    isMan: '',
                    pageSize: 10,
                    pageNumber: 1
                },
                dataCount: 0,
                loading: false,
                teacherId: 1,
                teacherName: "",
                tableData: []
            }
        },
        methods: {
            getTeacher() {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${teacherQuery}`,
                    variables: {
                        "classId": this.classId
                    }
                })
                .then(res => {
                    _this.teacherId = res.data.data.teachers[0].id;
                    _this.teacherName = res.data.data.teachers[0].name;
                });
            },
            getClassDetail() {
                let _this = this;
                this.tableData = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${classQuery}`,
                    variables: {
                        "id": this.classId
                    }
                })
                .then(res => {
                    _this.info = res.data.data.class;
                });
            },
            //获取学生列表
            searchStudents() {
                let _this =this;
                let params = {
                    "classId": this.classId,
                    "pageSize": this.filters.pageSize,
                    "pageNumber": this.filters.pageNumber

                };
                if (_this.filters.name !== '') {
                    params.name = _this.filters.name
                }
                if (_this.filters.studentNo !== '') {
                    params.studentNo = _this.filters.studentNo
                }
                if (_this.filters.isMan !== '') {
                    params.isMan = _this.filters.isMan
                }
                this.loading = true;
                this.tableData = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${studentQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.loading = false;
                    _this.dataCount = res.data.data.students.dataCount;
                    res.data.data.students.data.forEach(student => {
                        student.sex = student.isMan ? "男" : "女";
                        _this.tableData.push(student);
                    });
                })
            },
            goScore() {
                this.$router.push({ path: '/score/' + this.classId });
            },
            goData() {
                this.$router.push({ path: '/data/' + this.classId });
            },
            goStudentDetail(id) {
                this.$router.push({name: '学生信息详情', params: {id: id, class_id: this.classId} });
            },
            goTeacherDetail(teacherId) {
                console.log('工号', teacherId);
                this.$router.push({ path: '/teacherdetail/' + teacherId });
            },
            openSchedule() {
                console.log('查看班级课表');
                this.dialogTableVisible = true;
            },
            checkClassPhysical(classId){
                this.$router.push({
                    path: '/physicaltest/' + classId
                });
            }
        },
        mounted: function () {
            this.searchStudents();
            this.getClassDetail();
            this.getTeacher();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        min-width: 700px;
        .panel {
            border: 1px solid #d4d4d4;
            padding: 5px 15px 15px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        .table-panel {
            margin-top: 20px;
            min-height: 580px;
        }
        .title {
            line-height: 2.5;
            font-weight: bold;
            font-size: 14px;
        }
        .large{
            line-height: 2.5;
            font-weight: bold;
            font-size: 16px;
        }
        .sub-title {
            line-height: 2.5;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
        .pointer {
            cursor: pointer;
        }
        .class-teacher {
            text-align: center;
            span {
                display: block;
                font-weight: bold;
                line-height: 3;
            }
        }
        .score-panel {
            padding: 5px 15px 0;
        }
        .score {
            span {
                display: inline-block;
                width: 100px;
                line-height: 3;
                padding: 10px;
                cursor: pointer;
                text-align: center;
                margin-right: 20px;
            }
            i {
                width: 55px;
                height: 55px;
                background-color: #29b6f6;
                display: inline-block;
                text-align: center;
                line-height: 55px;
                border-radius: 50%;
                color: #fff;
                font-size: 28px;
            }
            .fa-database {
                background-color: #8bc34a;
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
    }
</style>