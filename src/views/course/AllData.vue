<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
            <el-breadcrumb-item>查看体测数据</el-breadcrumb-item>
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
                                <el-option v-for="term in options.terms" :key="term.id" :label="term.name" :value="term.id"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="search">筛选</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>

                <el-table :data="tableData" style="width: 100%" v-loading="loading" element-loading-text="玩命加载中">
                    <el-table-column prop="name" label="姓名" width="180">
                    </el-table-column>
                    <el-table-column prop="studentNo" label="学号" width="180">
                    </el-table-column>
                    <el-table-column prop="height" label="身高(cm)">
                    </el-table-column>
                    <el-table-column prop="weight" label="体重(kg)" width="180">
                    </el-table-column>
                    <el-table-column prop="lungCapacity" label="肺活量(ml)" width="180">
                    </el-table-column>
                    <el-table-column prop="bmi" label="BIM指数" width="180">
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
    // 获取体测数据
    const dataQuery = `
        query(
            $classId: Long
            $name: String
            $studentNo: String
            $isMan: Boolean
            $pageNumber: Int
            $pageSize: Int
        ){
            allData:searchStudents(
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
                    fitnessCheckDatas{
                        termId
                        height
                        weight
                        lungCapacity
                        bmi
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
            //获取列表
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

                this.getData(params);
            },
            formatData(allData, params) {
                let _this = this;
                _this.tableData = [];
                this.dataCount = allData.dataCount;
                allData.data.forEach(item => {
                    let listItem = {
                        studentNo: '',
                        name: '',
                        height: 0,
                        weight: 0,
                        lungCapacity: 0,
                        bmi: 0
                    };
                    listItem.name = item.name;
                    listItem.studentNo = item.studentNo;
                    listItem.height = item.fitnessCheckDatas[0].height;
                    listItem.weight = item.fitnessCheckDatas[0].weight;
                    listItem.lungCapacity = item.fitnessCheckDatas[0].lungCapacity;
                    listItem.bmi = item.fitnessCheckDatas[0].bmi;
                    _this.tableData.push(listItem);
                });
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
            getData(params) {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${dataQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.loading = false;
                    _this.formatData(res.data.data.allData, params);
                });
            }
        },
        mounted: function () {
            let params = {
                "pageSize": 10,
                "pageNumber": 1
            }
            this.getData(params);
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
        .filters{
            margin-top: 10px;
        }
        .title,
        .filters{
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