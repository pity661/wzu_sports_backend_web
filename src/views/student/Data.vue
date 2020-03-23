<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/students' }">学生管理</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/students' }">班级列表</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/classdetail/' + classId }">班级详情</el-breadcrumb-item>
            <el-breadcrumb-item>班级体测数据</el-breadcrumb-item>
        </el-breadcrumb>
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
                            <el-button type="primary" @click="searchStudent">筛选</el-button>
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
                    <el-pagination @current-change="searchStudent" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'

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
                classId: this.$route.params.class_id,
                options:{
                    terms: []
                },
                filters: {
                    "termId": '',
                    "classId": '',
                    "name": '',
                    "studentNo": '',
                    "isMan": '',
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
            searchStudent() {
                let _this = this;
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "classId": this.classId
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
            this.searchStudent();
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
    }
</style>