<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
            <el-breadcrumb-item>查看体育成绩</el-breadcrumb-item>
        </el-breadcrumb>
        <el-col :span="24" class="filters">
            <el-form :inline="true" :model="filters">
                <el-form-item label="学院">
                    <el-select class="filter-college" v-model="filters.college" value-key="id" placeholder="学院" @change="selectCollege">
                        <el-option v-for="item in options.colleges" :key="item.id" :label="item.name" :value="item"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="专业">
                    <el-select class="filter-major" v-model="filters.major" value-key="id" placeholder="专业" @change="selectMajor">
                        <el-option v-for="item in filters.college.majors" :key="item.id" :label="item.name" :value="item"></el-option>
                    </el-select>
                    
                </el-form-item>
                <el-form-item label="年级">
                    <el-select class="filter-grade" v-model="filters.grade" value-key="id" placeholder="年级" @change="getClasses">
                        <el-option v-for="item in options.grades" :key="item.id" :label="item.name" :value="item"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="班级">
                    <el-select class="filter-grade" v-model="filters.classId" placeholder="班级">
                        <el-option v-for="item in options.classes" :key="item.id" :label="item.name" :value="item.id"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
        </el-col>
        <div class="main-panel">
            <el-col class="table-panel panel">
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
                            <el-select class="filter-sex" v-model="filters.termId" placeholder="选择学期">
                                <el-option v-for="item in options.terms" :key="item.id" :label="item.name" :value="item.id"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="search">筛选</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>

                <el-table :data="tableData" v-loading="loading" element-loading-text="玩命加载中">
                    <el-table-column prop="name" label="姓名" width="90">
                    </el-table-column>
                    <el-table-column prop="studentNo" label="学号" width="100">
                    </el-table-column>
                    <el-table-column prop="sex" label="性别" width="90">
                    </el-table-column>
                    <el-table-column prop="termId" label="学期" width="100">
                    </el-table-column>
                    <el-table-column prop="meter50SprintTime" label="50米（秒）" width="100">
                    </el-table-column>
                    <el-table-column prop="meter50SprintScore" label="得分" width="90">
                    </el-table-column>
                    <el-table-column prop="standingJumpDistance" label="立定跳远（厘米）" width="100">
                    </el-table-column>
                    <el-table-column prop="standingJumpScore" label="得分" width="90">
                    </el-table-column>
                    <el-table-column prop="meter1500RunTime" label="1500米（秒）" width="100">
                    </el-table-column>
                    <el-table-column prop="meter1500RunScore" label="得分" width="90">
                    </el-table-column>
                    <el-table-column prop="abdominalCurlCount" label="仰卧起坐（个）" width="100">
                    </el-table-column>
                    <el-table-column prop="abdominalCurlScore" label="得分" width="90">
                    </el-table-column>
                </el-table>

                <div class="page">
                    <el-pagination @current-change="search" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'
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
    const classesQuery = `
    query ($majorId: Long $grade: Int) {
        classes(majorId: $majorId grade: $grade) {
        id
        name
        studentsCount
      }
    }`;
    const termsQuery = `
    query ($universityId: Long) {
      terms(universityId: $universityId) {
        id
        name
      }
    }`;
    // 获取体育成绩数据
    const scoreQuery = `
        query(
            $classId: Long
            $name: String
            $studentNo: String
            $isMan: Boolean
            $pageNumber: Int
            $pageSize: Int
        ){
            allScore:searchStudents(
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
                    name
                    studentNo
                    isMan
                    sportScores{
                        termId
                        meter50SprintTime
                        meter50SprintScore
                        standingJumpDistance
                        standingJumpScore
                        meter1500RunTime
                        meter1500RunScore
                        abdominalCurlCount
                        abdominalCurlScore
                    }
                }
            }
        }
    `;
    export default {
        data() {
            return {
                universityId: resources.universityId,
                options:{
                    colleges: [],
                    majors: [],
                    grades: this.getGrades(),
                    classes: [],
                    terms: []
                },
                filters: {
                    "termId": '',
                    "classId": '',
                    "name": '',
                    "studentNo": '',
                    "isMan": '',
                    "college": '',
                    "major": '',
                    "grade": ''
                },
                tableData: [],
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                loading: true
            }
        },
        methods: {
            //获取学生列表
            search() {
                let _this = this;
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber
                };
                if (_this.filters.classId !== '') {
                    params.classId = _this.filters.classId
                }
                if (_this.filters.name !== '') {
                    params.name = _this.filters.name
                }
                if (_this.filters.studentNo !== '') {
                    params.studentNo = _this.filters.studentNo
                }
                if (_this.filters.isMan !== '') {
                    params.isMan = _this.filters.isMan
                }
                if (_this.filters.termId !== '') {
                    params.termId = _this.filters.termId
                }

                this.getScore(params);
            },
            selectCollege() {
                this.filters.major = this.filters.college.majors[0];
            },
            selectMajor() {
                this.getClasses();
            },
            getColleges() {
                let _this = this;
                let params = {
                    "universityId": this.universityId
                }
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${collegesQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.options.colleges = res.data.data.colleges;
                });
            },
            getTerms() {
                let _this = this;
                let params = {
                    "universityId": this.universityId
                }
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${termsQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.options.terms = res.data.data.terms;
                });
            },
            getClasses() {
                let params = {
                    "majorId": this.filters.major.id
                }
                if (this.filters.grade != '') {
                    params.grade = this.filters.grade;
                }
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${classesQuery}`,
                    variables: params
                })
                .then(res => {
                    this.options.classes = res.data.data.classes
                    this.loading = false;
                });
            },
            // 生成年级
            getGrades() {
                var date = new Date();
                var currentYear = date.getFullYear();
                if (date.getMonth() <= 8) {
                    return [currentYear - 1, currentYear - 2, currentYear - 3, currentYear - 4]
                } else {
                    return [currentYear, currentYear - 1, currentYear - 2, currentYear - 3]
                }
            },
            getScore(params) {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${scoreQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.loading = false;
                    _this.formatData(res.data.data.allScore);
                });
            },
            formatData(allScore) {
                let _this = this;
                this.tableData = [];
                this.dataCount = allScore.dataCount;
                allScore.data.forEach(item => {
                    let listItem = {
                        studentNo: '',
                        name: '',
                        sex: '',
                        termId: '',
                        meter50SprintTime: 8.42,
                        meter50SprintScore: 72,
                        standingJumpDistance: 1.78,
                        standingJumpScore: 40,
                        meter1500RunTime: 451,
                        meter1500RunScore: 40,
                        abdominalCurlCount: 20,
                        abdominalCurlScore: 15
                    };
                    listItem.name = item.name;
                    listItem.sex = item.isMan ? '男' : '女';
                    listItem.studentNo = item.studentNo;
                    listItem.termId = item.sportScores[0].termId;
                    listItem.meter50SprintScore = item.sportScores[0].meter50SprintScore;
                    listItem.meter50SprintTime = item.sportScores[0].meter50SprintTime;
                    listItem.abdominalCurlCount = item.sportScores[0].abdominalCurlCount;
                    listItem.abdominalCurlScore = item.sportScores[0].abdominalCurlScore;
                    listItem.meter1500RunScore = item.sportScores[0].meter1500RunScore;
                    listItem.meter1500RunTime = item.sportScores[0].meter1500RunTime;
                    listItem.standingJumpDistance = item.sportScores[0].standingJumpDistance;
                    listItem.standingJumpScore = item.sportScores[0].standingJumpScore;
                    _this.tableData.push(listItem);
                });
            }
        },
        mounted: function () {
            let params = {
                "pageSize": 10,
                "pageNumber": 1
            }
            this.getScore(params);
            this.getColleges();
            this.getTerms();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .panel {
            border: 1px solid #d4d4d4;
            padding: 5px 15px 15px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        .filters {
            margin-top: 10px;
        }
        .title,
        .filters {
            line-height: 2.5;
            font-weight: bold;
            font-size: 14px;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
    }
</style>