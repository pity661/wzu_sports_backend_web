<template>
    <div class="page-container">
        <div class="main-panel">
            <el-col class="table-panel">
                <el-col :span="24">
                    <el-form :inline="true" :model="filters">
                        <el-form-item label="学生学号">
                            <el-input v-model="filters.studentNo" placeholder="输入学生学号"></el-input>
                        <el-form-item label="学生姓名">
                            <el-input v-model="filters.studentName" placeholder="输入学生姓名"></el-input>
                        </el-form-item>
                        </el-form-item>
                        <el-form-item label="班级">
                            <el-input v-model="filters.className" placeholder="输入学生班级"></el-input>
                        </el-form-item>
                        <!-- <el-form-item label="性别">
                            <el-select class="sm" v-model="filters.isMan" placeholder="性别">
                                <el-option label="男" value="true"></el-option>
                                <el-option label="女" value="false"></el-option>
                            </el-select>
                        </el-form-item> -->

                        

                        <el-form-item label="是否激活">
                            <el-select class="sm" v-model="filters.isUser" placeholder="是否激活">
                                <el-option label="全部" value="0"></el-option>
                                <el-option label="已激活" value="1"></el-option>
                                <el-option label="未激活" value="2"></el-option>
                            </el-select>
                        </el-form-item>
                        <br>
                        <el-form-item label="打卡次数">
                            <el-select class="sm" v-model="filters.signInCountOperator" placeholder="打卡次数">
                                <el-option label=">" value="GREATER_THAN"></el-option>
                                <el-option label="<" value="LESS_THAN"></el-option>
                                <el-option label="=" value="EQUAL"></el-option>
                                <el-option label="介于" value="BETWEEN"></el-option>
                            </el-select>
                        </el-form-item>

                        <el-form-item>
                            <el-input v-model="filters.signInCount" placeholder="请输入数值"></el-input>
                            <el-input v-if="filters.signInCountOperator === 'BETWEEN'" v-model="filters.anotherSignInCount" placeholder="请输入数值"></el-input>
                        </el-form-item>

                        <el-form-item>
                            <el-button type="primary" @click="searchStudents">筛选</el-button>
                        </el-form-item>

                        <div v-if="dataCount != -1" class="activation">
                            学生总数：{{dataCount}}人
                        </div>
                    </el-form>
                    
                </el-col> 

                <el-table :data="studentList"> 
                    <el-table-column prop="studentNo" label="学号">
                    </el-table-column>
                    <el-table-column prop="studentName" label="姓名">
                    </el-table-column>
                    <el-table-column label="性别">
                        <template scope="scope">
                            <span>{{scope.row.isMan ? '男' : '女'}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="className" label="班级">
                    </el-table-column>
                    <el-table-column label="是否已激活">
                        <template scope="scope">
                            <span>{{scope.row.openId == "" ? '未激活' : '已激活'}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="signInCount" label="打卡次数">
                    </el-table-column>

                    <el-table-column prop="runningActivityCount" label="跑步运动次数">
                        <template scope="scope">
                            <div v-if="scope.row.runningActivityCount == 0">
                            {{scope.row.runningActivityCount}}
                            </div>
                            <div v-else>
                            <el-button type="text" @click="getActivityStatistic(scope.row.studentId, 0)">{{scope.row.runningActivityCount}}</el-button>
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column prop="areaActivityCount" label="区域运动次数">
                        <template scope="scope">
                            <div v-if="scope.row.areaActivityCount == 0">
                            {{scope.row.areaActivityCount}}
                            </div>
                            <div v-else>
                            <el-button type="text" @click="getActivityStatistic(scope.row.studentId, 1)">{{scope.row.areaActivityCount}}</el-button>
                            </div>
                        </template>
                    </el-table-column>

                    <el-table-column  prop="runningActivityQualifiedCount" label="跑步运动达标次数">
                        <template scope="scope">
                            <div class="qualified" v-if="scope.row.runningActivityQualifiedCount > 0">
                                {{scope.row.runningActivityQualifiedCount}}
                            </div>
                            <div v-else>
                                {{scope.row.runningActivityQualifiedCount}}
                            </div>
                        </template>
                    </el-table-column>
                    <el-table-column prop="areaActivityQualifiedCount" label="区域运动达标次数">
                        <template scope="scope">
                            <div class="qualified" v-if="scope.row.areaActivityQualifiedCount > 0">
                                {{scope.row.areaActivityQualifiedCount}}
                            </div>
                            <div v-else>
                                {{scope.row.areaActivityQualifiedCount}}
                            </div>
                        </template>
                    </el-table-column>

                </el-table>
                <div class="page">
                    <el-pagination @current-change="searchStudents" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'

    // 获取轨迹
    const userQuery = `
    query(
        $universityId: Long
        $pageSize: Int
        $pageNumber: Int
        $isUser: Int
        $studentName: String
        $studentNo: String
        $className: String
        $signInCountOperator: Operator
        $signInCount: Long
        $anotherSignInCount: Long){
        allstudent:university(id: $universityId){
            stuInfo:studentStatisticInfo(
            isUser: $isUser
            universityId: $universityId
            pageSize: $pageSize
            pageNumber: $pageNumber
            studentName: $studentName
            studentNo: $studentNo
            className: $className
            signInCountOperator: $signInCountOperator
            signInCount: $signInCount
            anotherSignInCount: $anotherSignInCount){
                pageNum
                dataCount
                pageSize
                data{
                    className
                    openId
                    studentId
                    studentNo
                    studentName
                    isMan
                    signInCount
                    runningActivityCount
                    runningActivityQualifiedCount
                    areaActivityCount
                    areaActivityQualifiedCount
                }
            }
        }       
    }
    `
    export default {
        data() {
            return {
                isUser: this.$route.params.is_user,
                universityId: resources.universityId,
                pageSize: 10,
                pageNumber: 1,
                dataCount: -1,
                loading: false,
                options: {
                    project: null
                },
                filters: {
                    isMan: '',
                    studentName: '',
                    studentNo: '',
                    className: '',
                    isUser: '',
                    signInCountOperator: '',
                    signInCount: '',
                    anotherSignInCount: ''
                },
                studentList: [],
            }
        },
        methods: {
            searchStudents() {
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "universityId": this.universityId
                };
                let _this = this;
                this.filters.isUser == '' ? this.filters.isUser = _this.isUser : _this.isUser = this.filters.isUser;
                params.isUser = _this.filters.isUser;
                if (this.filters.studentName !== '') {
                    params.studentName = this.filters.studentName
                }
                // if (this.filters.isMan !== '') {
                //     params.isMan = this.filters.isMan
                // }
                if (this.filters.studentNo !== '') {
                    params.studentNo = this.filters.studentNo
                }
                if (this.filters.className !== '') {
                    params.className = this.filters.className
                }
                if (this.filters.signInCountOperator !== '') {
                    params.signInCountOperator = this.filters.signInCountOperator
                }
                if (this.filters.signInCount !== '') {
                    params.signInCount = this.filters.signInCount
                }
                if (this.filters.anotherSignInCount !== '') {
                    params.anotherSignInCount = this.filters.anotherSignInCount
                }
                this.getData(params);
            },
            getData(params) {
                let _this = this;
                this.studentList = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${userQuery}`,
                    variables: params
                })
                    .then(res => {
                        _this.loading = false;
                        _this.dataCount = res.data.data.allstudent.stuInfo.dataCount;
                        _this.studentList = res.data.data.allstudent.stuInfo.data;
                        // console.log(res.data.data.allstudent.stuInfo.data);
                    });
            },
            getActivityStatistic(studentId, type){
                console.log(studentId + "--" + type);
                this.$router.push({ path: '/activityDataStatistic/' + studentId + "&" + type});
            },
        },
        mounted: function () {
            this.searchStudents()
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        min-width: 1200px;
        .sm {
            width: 105px;
        }
        .table-panel {
            margin-top: 20px;
        }
        .title {
            line-height: 2.5;
            font-weight: bold;
            font-size: 14px;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
        .el-input {
            width: 145px;
        }
        .el-date-editor {
            width: 280px;
        }
        .pointer {
            cursor: pointer;
            color: #29b6f6;
        }
        .success {
            color: #13CE66;
        }
        .error {
            color: #FF4949;
        }
        #container {
            width: 650px;
            height: 500px;
            margin: 0px;
        }
        .qualified {
            color: #13CE66;
        }
        .activation{
            display:inline;
            margin-left: 10px;
            vertical-align: middle;
            display: inline-block;
            text-align: center;
            line-height: 35px;
            font-size: 20px;
            font-weight: bold;
            background-color: #f2f2f2;
            border: 1px solid #d4d4d4;
            border-bottom-color: transparent;
        }
    }
</style>
