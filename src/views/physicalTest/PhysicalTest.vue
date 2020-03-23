<template>
    <div class="page-container">
        <div class="main-panel">    
            <div class="table-panel">
                <!-- 筛选条件 -->
                <el-form :inline="true" :model="filters">
                    <el-form-item label="姓名">
                        <el-input v-model="filters.studentName" placeholder="输入学生姓名"></el-input>
                    </el-form-item>
                    <el-form-item label="学号">
                        <el-input v-model="filters.studentNo" placeholder="输入学号"></el-input>
                    </el-form-item>
                    <el-form-item label="班级">
                        <el-input v-model="filters.className" placeholder="输入学生班级"></el-input>
                    </el-form-item>
                    <el-form-item label="体测结果">
                        <el-select class="sm" v-model="filters.testResult" clearable placeholder="体测结果">
                            <el-option label="没测" value="1"></el-option>
                            <el-option label="免测" value="2"></el-option>
                            <el-option label="不及格" value="3"></el-option>
                        </el-select>
                    </el-form-item>
                    <div style="float:right">
                    <el-form-item v-if="showClassSelectButton" >
                        <el-button type="primary" @click="getPhysicalTest">在该班级中筛选</el-button>
                    </el-form-item>
                    <el-form-item label="EXCEL批量导入：">
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="toImportPhysicalTest">导入体测记录</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="toImportSportsCourse">导入教学班信息</el-button>
                    </el-form-item>
                    </div>
                    <br>
                    <el-form-item label="学年">
                        <el-select class="sm" v-model="filters.schoolYear" placeholder="学年" value-key="id" clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.schoolYear" :key="item.id" :label="item.schoolYear" :value="item.schoolYear"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="学期">
                        <el-select class="sm" v-model="filters.term" placeholder="学期" clearable v-on:change="getSelectCondition">
                            <el-option label="第一学期" :value="1"></el-option>
                            <el-option label="第二学期" :value="2"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="教师姓名">
                        <el-select class="sm" v-model="filters.teacherName" placeholder="教师姓名" filterable clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.teacherName" :key="item.id" :label="item.teacherName" :value="item.teacherName"></el-option>
                        </el-select>
                    </el-form-item>
                    <!-- <el-form-item label="教师姓名">
                        <el-autocomplete class="inline-input" v-model="filters.teacherName" :fetch-suggestions="querySearch" placeholder="教师姓名" @select="getSelectCondition"></el-autocomplete>
                    </el-form-item> -->
                    <el-form-item label="课程名">
                        <el-select class="sm" v-model="filters.courseName" placeholder="课程名" value-key="id" filterable clearable v-on:change="getSelectCondition">
                            <el-option v-for="item in options.courseName" :key="item.id" :label="item.courseName" :value="item.courseName"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="课程时间">
                        <el-select class="sm" v-model="filters.courseTime" filterable clearable placeholder="课程时间">
                            <el-option v-for="item in options.courseTime" :key="item.id" :label="item.courseTime" :value="item.courseTime"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="getNewPhysicalTest">筛选所有</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="addDate">新增体测记录</el-button>
                    </el-form-item>
                </el-form>
                <!-- 筛选条件 end -->
                <!-- 显示体测记录列表 -->
                <el-table :data="physicalList" v-loading="loading" element-loading-text="玩命加载中">
                    <el-table-column prop="studentNo" label="学号" width="150" fixed>
                    </el-table-column>
                    <el-table-column prop="studentName" label="姓名" fixed>
                        <template scope="scope">
                            <el-button type="text" @click="alterDate(scope.row.id)">{{scope.row.studentName}}</el-button>
                        </template>
                    </el-table-column>
                    <el-table-column prop="collegeName" label="学院" width="120">
                    </el-table-column>
                    <el-table-column prop="className" label="班级" width="150">
                    </el-table-column>
                    <el-table-column label="性别">
                        <template scope="scope">
                            <span>{{scope.row.isMan ? "男" : "女"}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="totalScore" label="总分">
                    </el-table-column>
                    <el-table-column prop="height" label="身高(cm)" width="100">
                    </el-table-column>
                    <el-table-column prop="weight" label="体重(kg)" width="100">
                    </el-table-column>
                    <el-table-column prop="vitalCapacity" label="肺活量(ml)" width="130">
                    </el-table-column>
                    <el-table-column prop="standingLongJump" label="立定跳远(cm)" width="130">
                    </el-table-column>
                    <el-table-column prop="sitAndReach" label="坐位体前屈(cm)" width="140">
                    </el-table-column>
                    <el-table-column label="仰卧起坐(个/分)/引体向上(个)" width="150">
                        <template scope="scope">
                            <span>{{scope.row.isMan ? scope.row.pullUp : scope.row.oneMinuteSitUp}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="fiftyRunTime" label="50米跑(s)" width="130">
                    </el-table-column>
                    <el-table-column label="800米/1000米(min)" width="160">
                        <template scope="scope">
                            <span>{{scope.row.isMan ? scope.row.oneThousandRunTime : scope.row.eightHundredRunTime}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="isMan" label="备注" width="120">
                    </el-table-column>
                </el-table>
                <div class="page">
                    <el-pagination @current-change="getPhysicalTest" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
                <!-- 显示体测记录列表 end -->

            </div>  
        </div>

        <el-dialog title="编辑体测信息" :visible.sync="showDialog">
            <el-form :model="form" :inline="true">     
                <el-form-item label="姓名">
                    <el-input v-model="form.studentName" placeholder="输入学生姓名"></el-input>
                </el-form-item>
                <el-form-item label="学号">
                    <el-input v-model="form.studentNo" placeholder="输入学号"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                    <el-select class="sm" v-model="form.isMan" placeholder="性别">
                        <el-option label="男" :value="true"></el-option>
                        <el-option label="女" :value="false"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="学院">
                    <el-input v-model="form.collegeName" placeholder="输入学院"></el-input>
                </el-form-item>
                <el-form-item label="班级">
                    <el-input v-model="form.className" placeholder="输入班级"></el-input>
                </el-form-item>
                <el-form-item label="身高">
                    <el-input v-model="form.height" placeholder="输入身高(cm)"></el-input>
                </el-form-item>
                <el-form-item label="体重">
                    <el-input v-model="form.weight" placeholder="输入体重(kg)"></el-input>
                </el-form-item>
                <el-form-item label="肺活量">
                    <el-input v-model="form.vitalCapacity" placeholder="输入肺活量(ml)"></el-input>
                </el-form-item>
                <el-form-item label="立定跳远">
                    <el-input v-model="form.standingLongJump" placeholder="立定跳远(cm)"></el-input>
                </el-form-item>
                <el-form-item label="坐位体前屈">
                    <el-input v-model="form.sitAndReach" placeholder="坐位体前屈(cm)"></el-input>
                </el-form-item>
                <el-form-item v-if="!form.isMan" label="一分钟仰卧起坐">
                    <el-input v-model="form.oneMinuteSitUp" placeholder="仰卧起坐(个/分)"></el-input>
                </el-form-item>
                <el-form-item v-if="form.isMan" label="引体向上">
                    <el-input v-model="form.pullUp" placeholder="引体向上(个)"></el-input>
                </el-form-item>
                <el-form-item label="50米跑">
                    <el-input v-model="form.fiftyRunTime" placeholder="50米跑成绩(s)"></el-input>
                </el-form-item>
                <el-form-item v-if="!form.isMan" label="800米跑">
                    <el-input v-model="form.eightHundredRunTime" placeholder="800米跑成绩(min)"></el-input>
                </el-form-item>
                <el-form-item v-if="form.isMan" label="1000米跑">
                    <el-input v-model="form.oneThousandRunTime" placeholder="1000米跑成绩(min)"></el-input>
                </el-form-item>
                <br>
                <el-form-item label="备    注：">
                    <el-input type="textarea" v-model="form.remarks" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="showDialog = false">取 消</el-button>
                <el-button type="primary" @click="checkFormMes">确 定</el-button>
            </div>

        </el-dialog>
    </div>
</template>

<script>
    import resources from '../../resources'
    //根据条件信息筛选体测记录
    const coursePhysicalQuery = `
        query(
            $universityId: Long
            $pageNumber: Int
            $pageSize: Int
            $className: String
            $studentName: String
            $studentNo: String
            $testResult: Int
            $schoolYear: String
            $term: Int
            $teacherName: String
            $courseName: String
            $courseTime: String
            $classId: Int
        ){
            allstudents:searchCoursePhysicalTest(
                universityId: $universityId
                pageNumber: $pageNumber
                pageSize: $pageSize
                className: $className
                studentName: $studentName
                studentNo: $studentNo
                testResult: $testResult
                schoolYear: $schoolYear
                term: $term
                teacherName: $teacherName
                courseName: $courseName
                courseTime: $courseTime
                classId: $classId
            ){
                pageNum
                pageSize
                pagesCount
                dataCount
                data {
                        id
                        isMan
                        studentNo
                        collegeName
                        studentName
                        className
                        totalScore
                        height
                        weight
                        vitalCapacity
                        standingLongJump
                        sitAndReach
                        oneMinuteSitUp
                        pullUp
                        fiftyRunTime
                        eightHundredRunTime
                        oneThousandRunTime
                    }
                }
            }
        }
    `;
    //教学班筛选条件
    const conditionQuery = `
        query(
            $universityId: Long
            $teacherName: String
            $courseName: String
            $schoolYear: String
            $term: Int
        ){
            university(
                id: $universityId
            ){
                teacherNameList(
                    schoolYear: $schoolYear
                    term: $term
                    ){
                    id
                    teacherName
                }
                schoolYearList{
                    id
                    schoolYear
                }
                courseNameList(
                    schoolYear: $schoolYear
                    term: $term
                    teacherName: $teacherName
                    ){
                    id
                    courseName
                }
                courseTimeList(
                    teacherName: $teacherName
                    courseName: $courseName
                    schoolYear: $schoolYear
                    term: $term
                    ){
                    id
                    courseTime
                }
            }
        }
    `;
    //修改或新增体测记录
    const addOrAlterPhysicalQuery = `
        query(
            $id: Long
            $universityId: Long
            $studentName: String
            $studentNo: String
            $isMan: Int
            $collegeName: String
            $className: String
            $height: Float
            $weight: Float
            $vitalCapacity: Long
            $standingLongJump: Int
            $sitAndReach: Float
            $oneMinuteSitUp: Int
            $pullUp: Int
            $fiftyRunTime: Float
            $eightHundredRunTime: Float
            $oneThousandRunTime: Float
            $remarks: String
            $userName: String
            $courseTime: String
            $courseName: String
            $teacherName: String
            $term: Int
            $schoolYear: String
        ){
            addOrAlterPhysicalTest(
                id: $id
                universityId: $universityId
                studentName: $studentName
                studentNo: $studentNo
                isMan: $isMan
                collegeName: $collegeName
                className: $className
                height: $height
                weight: $weight
                vitalCapacity: $vitalCapacity
                standingLongJump: $standingLongJump
                sitAndReach: $sitAndReach
                oneMinuteSitUp: $oneMinuteSitUp
                pullUp: $pullUp
                fiftyRunTime: $fiftyRunTime
                eightHundredRunTime: $eightHundredRunTime
                oneThousandRunTime: $oneThousandRunTime
                remarks: $remarks
                userName: $userName
                courseTime: $courseTime
                courseName: $courseName
                teacherName: $teacherName
                term: $term
                schoolYear: $schoolYear
            ){
                result
            }
        }
    `
    export default {
        data() {
            return {
                universityId: resources.universityId,
                classId: this.$route.params.classId,
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                loading: false,
                showClassSelectButton: false,
                options: {
                    teacherName: [],
                    schoolYear: [],
                    courseName: [],
                    courseTime: [],
                },
                filters: {
                    studentName: '',
                    studentNo: '',
                    className: '',
                    testResult: '',
                    schoolYear: '',
                    term: '',
                    teacherName: '',
                    courseName: '',
                    courseTime: ''
                },
                physicalList: [],
                teacherNameList: [],

                showDialog: false,
                form: {
                },
                result: '',
                mistakeMes: '',
            }
        },

        methods: {
            toImportSportsCourse() {
                this.$router.push({ path: '/importSportsCourse'});
            },
            //点击列表中学生姓名修改体测数据
            toImportPhysicalTest() {
                this.$router.push({ path: '/importPhysicalTest'});
            },
            alterDate(id){
                let _this = this;
                for (var i = 0; i < _this.physicalList.length; i++) {
                    if (_this.physicalList[i].id == id) {
                        this.form = _this.physicalList[i];
                        this.form.remarks = '';
                        break;
                    }
                }
                _this.showDialog = true;
            },
            //点击添加学生体测信息按钮
            addDate(){
                let _this = this;
                this.resetFormMes();
                if (_this.filters.schoolYear == '' || _this.filters.term == '' || _this.filters.schoolYear == null){
                    this.mistakeMes = "请先选择学年，学期后点击该按钮";
                    this.lackMessage(this.mistakeMes);
                } else {
                    if (_this.filters.studentName !== '') {
                        this.form.studentName = this.filters.studentName;
                    }
                    if (_this.filters.studentNo !== '' ) {
                        this.form.studentNo = this.filters.studentNo;
                    }
                    if (_this.filters.className !== '') {
                        this.form.className = this.filters.className;
                    }
                    if (_this.filters.schoolYear !== '' && _this.filters.schoolYear !== null) {
                        this.form.schoolYear = this.filters.schoolYear;
                    }
                    if (_this.filters.term !== '') {
                        this.form.term = this.filters.term;
                    }
                    if (_this.filters.teacherName !== '') {
                        this.form.teacherName = this.filters.teacherName;
                    }
                    if (_this.filters.courseName !== '' && _this.filters.courseName !== null) {
                        this.form.courseName = this.filters.courseName;
                    }
                    if (_this.filters.courseTime !== '' && _this.filters.courseTime !== null) {
                        this.form.courseTime = this.filters.courseTime;
                    }
                    _this.showDialog = true;
                }
            },
            //将修改和新增数据发到后台
            updateNewDate(){
                let _this = this;
                if (this.form.isMan == true){
                    this.form.isMan = 1;
                } else if (this.form.isMan == false){
                    this.form.isMan = 0;
                }
                this.form.userName = sessionStorage.getItem('userName');
                this.form.universityId = this.universityId;
                this.$ajax.post(`${resources.graphQlApi}`,{
                                'query': `${addOrAlterPhysicalQuery}`,
                                variables: this.form
                    })
                    .then(res => {
                        _this.result = res.data.data.addOrAlterPhysicalTest.result;
                        if (this.result === true) {
                            this.succeed();
                        } else {
                            this.mistake();
                        }
                    });
                console.log(this.form);
                this.showDialog = false;
            },
            //验证弹框输入信息
            checkFormMes(){
                let _this = this;
                _this.result = '';
                _this.mistakeMes = '';
                if (_this.form.id == '') {
                    delete this.form["id"];
                }
                if (_this.form.courseName == '') {
                    delete this.form["courseName"];
                }
                if (_this.form.teacherName == '') {
                    delete this.form["teacherName"];
                }
                if (_this.form.courseTime == '') {
                    delete this.form["courseTime"];
                }
                if (_this.form.studentName == '') {
                    _this.mistakeMes = _this.mistakeMes + "姓名、";
                    delete this.form["studentName"];
                }
                if (_this.form.studentNo == '') {
                    _this.mistakeMes = _this.mistakeMes + "学号、";
                    delete this.form["studentNo"];
                }
                if (_this.form.collegeName == '') {
                    _this.mistakeMes = _this.mistakeMes + "学院名、";
                    delete this.form["collegeName"];
                }
                if (_this.form.className == '') {
                    _this.mistakeMes = _this.mistakeMes + "班级、";
                    delete this.form["className"];
                }
                if (_this.form.height == '') {
                    delete this.form["height"];
                }
                if (_this.form.weight == '') {
                    delete this.form["weight"];
                }
                if (_this.form.vitalCapacity == '') {
                    delete this.form["vitalCapacity"];
                }
                if (_this.form.standingLongJump == '') {
                    delete this.form["standingLongJump"];
                }
                if (_this.form.sitAndReach == '') {
                    delete this.form["sitAndReach"];
                }
                if (_this.form.oneMinuteSitUp == '') {
                    delete this.form["oneMinuteSitUp"];
                }
                if (_this.form.pullUp == '') {
                    delete this.form["pullUp"];
                }
                if (_this.form.fiftyRunTime == '') {
                    delete this.form["fiftyRunTime"];
                }
                if (_this.form.eightHundredRunTime == '') {
                    delete this.form["eightHundredRunTime"];
                }
                if (_this.form.oneThousandRunTime == '') {
                    delete this.form["oneThousandRunTime"];
                }
                if (_this.form.remarks == '') {
                    delete this.form["remarks"];
                }
                if (_this.mistakeMes !== '') {      
                    _this.mistakeMes = _this.mistakeMes + "不能为空";
                    this.lackMessage(_this.mistakeMes);
                    this.showDialog = false;
                } else {
                    this.updateNewDate();
                }
            },
            //初始化弹框form表单数据 有个问题isMan在这里不能初始化，String类型和Boolean类型都试过了，搞不定
            resetFormMes(){
                this.form.id = '';
                this.form.universityId = '';
                this.form.schoolYear = '';
                this.form.term = '';
                this.form.teacherName = '';
                this.form.courseName = '';
                this.form.courseTime = '';
                this.form.studentName = '';
                this.form.studentNo = '';
                this.form.collegeName = '';
                this.form.className = '';
                this.form.height = '';
                this.form.weight = '';
                this.form.vitalCapacity = '';
                this.form.standingLongJump = '';
                this.form.sitAndReach = '';
                this.form.oneMinuteSitUp = '';
                this.form.pullUp = '';
                this.form.fiftyRunTime = '';
                this.form.eightHundredRunTime = '';
                this.form.oneThousandRunTime = '';
                this.form.remarks = '';
            },

            succeed(){
                this.$notify({
                    title: '成功',
                    message: '添加成功',
                    type: 'success'
                    });
            },
            lackMessage(mistakeMes){
                this.$notify({
                    title: '警告',
                    message: mistakeMes,
                    type: 'warning'
                    });
            },
            mistake(){
                this.$notify.error({
                    title: '错误',
                    message: '添加失败'
                    });
            },

            //输入教师名字自动匹配存在教师
            // querySearch(queryString, cb) {
            //     var teacherNames = this.teacherNameList;
            //     var results = queryString ? teacherNames.filter(this.createFilter(queryString)) : teacherNames;
            //     // 调用 callback 返回建议列表的数据
            //     cb(results);
            // },
            // createFilter(queryString) {
            //     return (teacherName) => {
            //         return (teacherName.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
            //     };
            // },
            //开始新的筛选，同时去除班级id
            getNewPhysicalTest() {
                let _this = this;
                _this.classId = -1;
                _this.showClassSelectButton = false;
                _this.pageNumber = 1;
                this.getPhysicalTest()
            },
            //根据筛选条件筛选学生体测记录
            getPhysicalTest() {
                let _this = this;
                let params = {
                    "universityId": this.universityId,
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber
                };
                if (typeof(this.classId) != 'undefined' && this.classId !== -1) {
                    params.classId = this.classId
                    _this.showClassSelectButton = true;
                } else {
                    _this.showClassSelectButton = false;
                }
                if (this.filters.studentName !== '') {
                    params.studentName = this.filters.studentName
                }
                if (this.filters.studentNo !== '') {
                    params.studentNo = this.filters.studentNo
                }
                if (this.filters.className !== '') {
                    params.className = this.filters.className
                }
                if (this.filters.testResult !== '') {
                    params.testResult = parseInt(this.filters.testResult)
                }
                if (this.filters.schoolYear !== '') {
                    params.schoolYear = this.filters.schoolYear
                }
                if (this.filters.term !== '') {
                    params.term = parseInt(this.filters.term)
                }
                if (this.filters.teacherName !== '') {
                    params.teacherName = this.filters.teacherName
                }
                if (this.filters.courseName !== '') {
                    params.courseName = this.filters.courseName
                }
                if (this.filters.courseTime !== '') {
                    params.courseTime = this.filters.courseTime
                }
                this.getDate(params)
            },
            getDate(params){
                let _this = this;
                this.loading = true;
                _this.physicalList = [];
                this.$ajax.post(`${resources.graphQlApi}`,{
                                'query': `${coursePhysicalQuery}`,
                                variables: params
                    })
                    .then(res => {
                        this.loading = false;
                        _this.dataCount = res.data.data.allstudents.dataCount;
                        this.physicalList = res.data.data.allstudents.data;
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
                    params.term = parseInt(this.filters.term)
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
                _this.teacherNameList = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                                'query': `${conditionQuery}`,
                                variables: params
                    })
                .then(res => {
                     _this.options.schoolYear = res.data.data.university.schoolYearList;
                    //  if (this.filters.schoolYear =='') {
                    //     this.filters.schoolYear = _this.options.schoolYear[0].schoolYear;
                    //  }
                     _this.options.teacherName = res.data.data.university.teacherNameList;
                     _this.options.courseName = res.data.data.university.courseNameList;
                     _this.options.courseTime = res.data.data.university.courseTimeList;
                     for (var i = 0;i < _this.options.teacherName.length;i ++) {
                         this.teacherNameList.push({
                             "value": this.options.teacherName[i].teacherName
                         })
                     }
                });
            }

        },
        mounted: function () {
            this.getSelectCondition(),
            this.getPhysicalTest()
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