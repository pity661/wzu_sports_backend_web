<template>
    <div class="page-container">
        <div class="main-panel">
            <el-col :span="20">
                <div class="activation">
                    学生总数：<el-button type="text" @click="getStudent(0)">{{allStudentNum}}</el-button>人
                </div>
                <div class="activation">
                    已激活学生数：<el-button type="text" @click="getStudent(1)">{{activationNum}}</el-button>人
                </div>
                <div class="activation">
                    未激活学生数：<el-button type="text" @click="getStudent(2)">{{unActivationNum}}</el-button>人
                </div>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="goPhysicalTest">查看学生体测数据</el-button>
            </el-col>
            <el-col :span="20">
                <el-form :inline="true" :model="filters">
                    <el-form-item label="学院">
                        <el-select class="filter-college" v-model="filters.college" value-key="id" v-on:change="selectCollege">
                            <el-option v-for="college in colleges" :key="college.id" :label="college.name" :value="college"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="专业">
                        <el-select class="filter-major" v-model="filters.major" value-key="id" v-on:change="selectMajor">
                            <el-option v-for="major in filters.college.majors" :key="major.id" :label="major.name" :value="major"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="年级">
                        <el-select class="filter-grade" v-model="filters.grade" v-on:change="getClasses" placeholder="年级">
                            <el-option v-for="grade in grades" :key="grade.id" :label="grade" :value="grade"></el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </el-col>

            <el-col :span="4">
                <el-form>
                    <el-form-item>
                        <el-button type="primary" @click="batchAddStudents">批量创建学生账号</el-button>
                    </el-form-item>
                </el-form>
            </el-col>

            <el-col :span="24" class="classes-panel" v-loading="loading" element-loading-text="玩命加载中">
                <div v-for="item in classes" class="card" @click="goDetail(item)">
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
    import gql from 'graphql-tag'
    import resources from '../../resources'

    const classesQuery = `
    query ($majorId: Long $grade: Int) {
        classes(majorId: $majorId grade: $grade) {
        id
        name
        studentsCount
      }
    }`;

    const collegesQuery = `
    query ($universityId: Long) {
      colleges(universityId: $universityId) {
        id
        name
        majors {
            id
            name
        }
      }
    }`;

    const studentNumQuery = `
    query ($universityId: Long){
        university(id: $universityId){
            activeStudentCount
            studentCount
        }
    }
    `;
    export default {

        data() {
            return {
                universityId: resources.universityId,
                allStudentNum: 0,
                activationNum: 0,
                unActivationNum: 0,
                colleges: [],
                total: 0,
                currentPage: 1,
                loading: true,
                classes: [],
                grades: this.getGrades(),
                filters: {
                    college: {},
                    major: {},
                    grade: ''
                }
            }
        },
        methods: {
            goPhysicalTest(){
                this.$router.push({ path: '/physicaltest' });
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
            goDetail(item) {
                this.$router.push({ path: '/classdetail/' + item.id });
            },
            selectCollege() {
                this.filters.major = this.filters.college.majors[0]
            },
            selectMajor() {
                this.getClasses();
            },
            getStudent(num){
                this.$router.push({ path: '/allstudent/' + num });
            },
            getStudentNum(){
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`,{
                    'query': `${studentNumQuery}`,
                    variables:{
                        "universityId": _this.universityId
                    }
                })
                .then(res => {
                    _this.allStudentNum = res.data.data.university.studentCount;
                    _this.activationNum = res.data.data.university.activeStudentCount;
                    _this.unActivationNum = _this.allStudentNum - _this.activationNum;
                })
            },
            getColleges() {
                let params = {
                    "universityId": this.universityId
                }
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${collegesQuery}`,
                    variables: params
                })
                .then(res => {
                    this.colleges = res.data.data.colleges
                    this.filters.college = res.data.data.colleges[0]
                    this.filters.major = res.data.data.colleges[0].majors[0]
                });
            },
            getClasses() {
                let params = {
                    "majorId": this.filters.major.id,
                    "universityId": this.universityId
                }
                if (this.filters.grade != '') {
                    params.grade = this.filters.grade;
                }
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${classesQuery}`,
                    variables: params
                })
                .then(res => {
                    this.loading = false;
                    this.classes = res.data.data.classes;
                });
            },
            getGrades() {
                var date = new Date()
                var currentYear = date.getFullYear()
                if (date.getMonth() <= 8) {
                    return [currentYear - 1, currentYear - 2, currentYear - 3, currentYear - 4]
                } else {
                    return [currentYear, currentYear - 1, currentYear - 2, currentYear - 3]
                }
            }
        },
        mounted: function () {
            this.getStudentNum();
            this.getColleges();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        .main-panel {
            overflow: hidden;
            margin-top: 10px;
            padding: 0 10px 0px 15px;
        }
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .filter-sex {
            width: 90px;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
        .activation{
            margin: 10px;
            text-align: center;
            width: 252px;
            height: 50px;
            line-height: 50px;
            font-size: 20px;
            font-weight: bold;
            background-color: #f2f2f2;
            border: 1px solid #d4d4d4;
            border-bottom-color: transparent;
            .el-button{
                font-size: 20px;
            }
        }
        .el-form-item{
            margin: 5px;
        }
        .activation-num{
            font-size: 18px;
        }
        .classes-panel {
            overflow: hidden;
            font-size: 14px;
            .card {
                display: inline-block;
                margin: 10px;
                text-align: center;
                cursor: pointer;
            }
            .classes-name {
                width: 252px;
                height: 165px;
                line-height: 165px;
                font-size: 23px;
                font-weight: bold;
                background-color: #f2f2f2;
                border: 1px solid #d4d4d4;
                border-bottom-color: transparent;
            }
            .student-number {
                width: 252px;
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
