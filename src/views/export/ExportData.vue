<template>
    <div class="page-container">
        <div class="main-panel">    
            <div class="table-panel">
                <el-form :inline="true" :model="filters">
                    <el-form-item label="学年">
                        <el-select class="sm" v-model="filters.schoolYear" placeholder="学年" clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.schoolYear" :label="item.schoolYear" :value="item.schoolYear"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="学期">
                        <el-select class="sm" v-model="filters.term" placeholder="学期" filterable clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.term" :label="item.term" :value="item.term"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="教师姓名">
                        <el-select class="sm" v-model="filters.teacherName" placeholder="教师姓名" filterable clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.teacherName" :label="item.teacherName" :value="item.teacherName"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="课程名">
                        <el-select class="sm" v-model="filters.courseName" placeholder="课程名" filterable clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.courseName" :label="item.courseName" :value="item.courseName"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="课程时间">
                        <el-select class="sm" v-model="filters.courseTime" filterable clearable placeholder="课程时间">
                            <el-option v-for="item in options.courseTime" :label="item.courseTime" :value="item.courseTime"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="exportFile()">导出</el-button>
                    </el-form-item>
                </el-form>
            </div>  
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'
    
    //教学班筛选条件
    const teachingClassQuery = `
        query(
            $universityId: Long
            $teacherName: String
            $courseName: String
            $schoolYear: String
            $term: String
        ){
            university(
                id: $universityId
            ){
                schoolYearList : getSchoolYear{
                    schoolYear
                }
                termList : getTerm(
                    schoolYear : $schoolYear
                    ){
                    term
                }
                teacherNameList : getTeacher(
                    schoolYear: $schoolYear
                    term: $term
                    ){
                    teacherName
                }
                courseNameList : getCourseName(
                    schoolYear: $schoolYear
                    term: $term
                    teacherName: $teacherName
                    ){
                    courseName
                }
                courseTimeList : getCourseTime(
                    teacherName: $teacherName
                    courseName: $courseName
                    schoolYear: $schoolYear
                    term: $term
                    ){
                    courseTime
                }
            }
        }
    `;
    
    export default {
        data() {
            return {
                universityId: resources.universityId,
                options: {
                    teacherName: [],
                    schoolYear: [],
                    courseName: [],
                    courseTime: [],
                    term: []
                },
                filters: {
                    schoolYear: '',
                    term: '',
                    teacherName: '',
                    courseName: '',
                    courseTime: ''
                }
            }
        },

        methods: {
            //导出文件
            exportFile(){
                // let url = resources.exportFile();
                let url = resources.host + "/exportFile";
                let params = new URLSearchParams();
                params.append('universityId',this.universityId);
                params.append('schoolYear', this.filters.schoolYear);
                params.append('term', this.filters.term);
                params.append('teacherName', this.filters.teacherName);
                params.append('courseName', this.filters.courseName);
                params.append('courseTime', this.filters.courseTime);
                this.$ajax.post(url, params)
                    .then(res => {
                        console.log(res);
                        location.href = resources.host + '/downloads/file?fileName=' + res.data;
                    });
            },

            //根据前置条件获取教学班筛选条件
            getSelectCondition(){
                let _this = this;
                let params = {
                    "universityId": this.universityId
                };
                if (this.filters.schoolYear !== '') {
                    params.schoolYear = this.filters.schoolYear
                }
                if (this.filters.term !== '') {
                    params.term = this.filters.term
                }
                if (this.filters.teacherName !== '') {
                    params.teacherName = this.filters.teacherName
                }
                if (this.filters.courseName !== ''){
                    params.courseName = this.filters.courseName
                }
                this.getCondition(params)
            },
            //教学班筛选条件获取
            getCondition(params){
                let _this = this;
                _this.options.schoolYear = [];
                _this.options.teacherName = [];
                _this.options.courseName = [];
                _this.options.courseTime = [];
                _this.options.term = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                                'query': `${teachingClassQuery}`,
                                variables: params
                    })
                .then(res => {
                     _this.options.schoolYear = res.data.data.university.schoolYearList;
                     _this.options.term = res.data.data.university.termList;
                     _this.options.teacherName = res.data.data.university.teacherNameList;
                     _this.options.courseName = res.data.data.university.courseNameList;
                     _this.options.courseTime = res.data.data.university.courseTimeList;
                });
            }

        },
        mounted: function () {
            this.getSelectCondition()
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
    }
</style>