<template>
    <div class="page-container">
        <div class="main-panel">
            <el-col class="table-panel">
                <el-col :span="24">
                    <el-form :inline="true" :model="filters">
                        <el-form-item label="学生姓名">
                            <el-input v-model="filters.studentName" placeholder="输入学生姓名"></el-input>
                        </el-form-item>
                        <el-form-item label="学生学号">
                            <el-input v-model="filters.studentNo" placeholder="输入学生学号"></el-input>
                        </el-form-item>
                        <el-form-item label="运动开始时间">
                            <el-date-picker v-model="filters.timeRange" type="datetimerange" placeholder="选择时间范围">
                            </el-date-picker>
                        </el-form-item>
                        <el-form-item label="运动项目">
                            <el-select class="sm" v-model="filters.runningSportId" placeholder="运动项目">
                                <el-option v-for="item in options.project" :key="item.id" :label="item.name" :value="item.id"></el-option>
                            </el-select>
                        </el-form-item>
                        <!-- <el-form-item label="性别">
                            <el-select class="sm" v-model="filters.isMan" placeholder="性别">
                                <el-option label="男" value="true"></el-option>
                                <el-option label="女" value="false"></el-option>
                            </el-select>
                        </el-form-item> -->
                        <el-form-item label="异常判断">
                            <el-select class="sm" v-model="filters.isValid" placeholder="异常判断">
                                <el-option label="全部" value="ALL"></el-option>
                                <el-option label="数据正常" value="true"></el-option>
                                <el-option label="数据异常" value="false"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="达标结果">
                            <el-select class="sm" v-model="filters.qualified" placeholder="达标结果">
                                <el-option label="全部" value="ALL"></el-option>
                                <el-option label="达标" value="true"></el-option>
                                <el-option label="未达标" value="false"></el-option>
                            </el-select>
                        </el-form-item>
                        <br>
                        <el-form-item label="平均速度">
                            <el-select class="sm" v-model="filters.speedOperator" placeholder="平均速度">
                                <el-option label=">" value="GREATER_THAN"></el-option>
                                <el-option label="<" value="LESS_THAN"></el-option>
                                <el-option label="=" value="EQUALL"></el-option>
                                <el-option label="介于" value="BETWEEN"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-input v-model="filters.speed" placeholder="请输入数值"></el-input>
                            <el-input v-if="filters.speedOperator === 'BETWEEN'" v-model="filters.anotherSpeed" placeholder="请输入数值"></el-input>
                        </el-form-item>
                        <el-form-item label="每秒步数">
                            <el-select class="sm" v-model="filters.stepPerSecondOperator" placeholder="每秒步数">
                                <el-option label=">" value="GREATER_THAN"></el-option>
                                <el-option label="<" value="LESS_THAN"></el-option>
                                <el-option label="=" value="EQUAL"></el-option>
                                <el-option label="介于" value="BETWEEN"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-input v-model="filters.stepPerSecond" placeholder="请输入数值"></el-input>
                            <el-input v-if="filters.stepPerSecondOperator === 'BETWEEN'" v-model="filters.anotherStepPerSecond" placeholder="请输入数值"></el-input>
                        </el-form-item>
                        <el-form-item label="平均步幅">
                            <el-select class="sm" v-model="filters.distancePerStepOperator" placeholder="平均步幅">
                                <el-option label=">" value="GREATER_THAN"></el-option>
                                <el-option label="<" value="LESS_THAN"></el-option>
                                <el-option label="=" value="EQUAL"></el-option>
                                <el-option label="介于" value="BETWEEN"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-input v-model="filters.distancePerStep" placeholder="请输入数值"></el-input>
                            <el-input v-if="filters.distancePerStepOperator === 'BETWEEN'" v-model="filters.anotherDistancePerStep" placeholder="请输入数值"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="search">筛选</el-button>
                        </el-form-item>
                        <el-form-item v-if="isShow">
                            <el-button type="primary" @click="dimSearch">模糊筛选</el-button>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="exportDate">导出</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>

                <el-table :data="studentList" style="width: 92%">
                    <el-table-column prop="" label="记录编号" >
                        <template scope="scope">
                            {{(allRecords.pageNum - 1) * pageSize + scope.$index + 1}}
                        </template>
                    </el-table-column>
                    <el-table-column prop="student.name" label="姓名">
                    </el-table-column>
                    <el-table-column prop="student.studentNo" label="学号" width="130">
                    </el-table-column>
                    <el-table-column label="性别">
                        <template scope="scope">
                            <span>{{scope.row.student.isMan ? '男' : '女'}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="project" label="运动项目" width="120">
                    </el-table-column>
                    <el-table-column prop="distance" label="距离(m)">
                    </el-table-column>
                    <el-table-column prop="costTime" label="耗时(s)">
                    </el-table-column>
                    <el-table-column prop="speed" label="平均速度(m/s)" width="130">
                    </el-table-column>
                    <el-table-column prop="stepCount" label="步数(步)">
                    </el-table-column>
                    <el-table-column prop="stepPerSecond" label="每秒步数(步)" width="120">
                    </el-table-column>
                    <el-table-column prop="distancePerStep" label="平均步幅(m)" width="120">
                    </el-table-column>
                    <el-table-column label="异常判断" width="120">
                        <template scope="scope">
                            <span :class="{ 'success': scope.row.isValid, 'error': !scope.row.isValid }">{{scope.row.isValid ? '数据正常' : '数据异常'}}</span>
                        </template>
                    </el-table-column>
                    <el-table-column label="达标结果" width="120">
                        <template scope="scope">
                            <span v-if="scope.row.endedAt" :class="{ 'success': scope.row.qualified, 'error': !scope.row.qualified }">{{scope.row.qualified ? '达标' : '未达标'}}</span>
                            <span v-if="!scope.row.endedAt" class="error">非正常结束</span>
                        </template>
                    </el-table-column>
                    <el-table-column label="运动轨迹" width="120">
                        <template scope="scope">
                            <el-button type="text" @click="getPath(scope.row.id)">查看轨迹</el-button>
                        </template>
                    </el-table-column>
                    <el-table-column prop="startTime" label="运动开始时间" width="170">
                    </el-table-column>
                </el-table>

                <div class="page">
                    <el-pagination @current-change="searchRecords" :current-page.sync="pageNumber" :page-size="pageSize" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>

        <el-dialog title="运动轨迹" :visible.sync="pathShow">
            <div id="container"></div>
        </el-dialog>
    </div>
</template>

<script>
    import resources from '../../resources';
    
    import qs from 'qs';
    // 运动方式列表
    const sportsQuery = `
    query($universityId: Long){
        runningSports(universityId:$universityId) {
            id
            name
        }
    }
    `;
    // 筛选运动记录
    const recordsQuery = `
    query(
        $universityId: Long
        $studentName: String
        $studentNo: String
        $startTime: Long
        $endTime: Long
        $runningSportId: Long
        $speed: Float
        $speedOperator: Operator
        $anotherSpeed: Float
        $stepPerSecond: Float
        $stepPerSecondOperator: Operator
        $anotherStepPerSecond: Float
        $distancePerStep: Float
        $distancePerStepOperator: Operator
        $anotherDistancePerStep: Float
        $pageNumber: Int
        $pageSize: Int
        $qualified: Boolean
        $isValid: Boolean
        $selectAll: Boolean
        ){
            allRecords:searchRunningActivities(
            universityId:$universityId
            studentName:$studentName
            studentNo:$studentNo
            startTime:$startTime
            endTime:$endTime
            runningSportId:$runningSportId
            speed:$speed
            speedOperator:$speedOperator
            anotherSpeed:$anotherSpeed
            stepPerSecond:$stepPerSecond
            stepPerSecondOperator:$stepPerSecondOperator
            anotherStepPerSecond:$anotherStepPerSecond
            distancePerStep:$distancePerStep
            distancePerStepOperator:$distancePerStepOperator
            anotherDistancePerStep:$anotherDistancePerStep
            pageNumber:$pageNumber
            pageSize:$pageSize
            qualified: $qualified
            isValid: $isValid
            selectAll: $selectAll
            ){
                pagesCount
                dataCount
                pageNum
                data{
                    id
                    runningSportId
                    distance
                    stepCount
                    costTime
                    startTime
                    speed
                    stepPerSecond
                    distancePerStep
                    qualified
                    isValid
                    endedAt
                    student{
                        name
                        studentNo
                        isMan
                    }
                    studentNo
                }
            }
        }
    `;

    // 获取轨迹
    const pathQuery = `
    query($id: Long){
        runningActivity(id:$id) {
            id
            runningSportId
            runningSport{
                name
            }
            data {
                longitude
                latitude
                isNormal
            }
        }
    }
    `
    export default {
        data() {
            return {
                loadingShow: null,

                universityId: resources.universityId,
                input: '',
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                loading: false,
                options: {
                    project: null
                },
                filters: {
                    // isMan: '',
                    studentName: '',
                    studentNo: '',
                    timeRange: ['', ''],
                    isValid: '',
                    qualified: '',
                    runningSportId: '',
                    speedOperator: '',
                    speed: '',
                    anotherSpeed: '',
                    stepPerSecondOperator: '',
                    stepPerSecond: '',
                    anotherStepPerSecond: '',
                    distancePerStepOperator: '',
                    distancePerStep: '',
                    anotherDistancePerStep: ''

                },
                studentList: [],
                allRecords: [],
                pathShow: false,
                pathDataOrigin: [],
                pathData: [],
                flagStudentNo: '',
                isShow: false,
                isClick: false
            }
        },
        methods: {
            // 获取运动方式下拉框
            getSports() {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${sportsQuery}`,
                    variables: {
                        "universityId": _this.universityId
                    }
                })
                    .then(res => {
                        _this.options.project = res.data.data.runningSports;
                    });
            },
            search() {
                this.isClick = false,
                this.searchRecords()
            },
            //获取列表
            dimSearch() {
                this.isClick = true;
                this.isShow = false;
                this.searchRecords()
            },
            exportDate() {
                let params = {
                    "universityId": this.universityId
                };
                if (this.filters.studentName !== '') {
                        params.studentName = this.filters.studentName
                    }
                    if (this.filters.studentNo !== '') {
                        params.studentNo = this.filters.studentNo
                    }
                    if (this.filters.timeRange[0] !== '' && this.filters.timeRange[0] !== null) {
                        params.startTime = this.filters.timeRange[0].getTime()
                    }
                    if (this.filters.timeRange[1] !== '' && this.filters.timeRange[1] !== null) {
                        params.endTime = this.filters.timeRange[1].getTime()
                    }
                    if (this.filters.runningSportId !== '') {
                        params.runningSportId = this.filters.runningSportId
                    }
                    if (this.filters.isValid !== '' && this.filters.isValid !== 'ALL') {
                        params.isValid = this.filters.isValid
                    }
                    if (this.filters.qualified !== '' && this.filters.qualified !== 'ALL') {
                        params.qualified = this.filters.qualified
                    }
                    if (this.filters.qualified !== '') {
                        params.qualified = this.filters.qualified
                    }
                    if (this.filters.speedOperator !== '') {
                        params.speedOperator = this.filters.speedOperator
                    }
                    if (this.filters.speed !== '') {
                        params.speed = parseFloat(_this.filters.speed);
                    }
                    if (this.filters.anotherSpeed !== '') {
                        params.anotherSpeed = parseFloat(_this.filters.anotherSpeed);
                    }
                    if (this.filters.stepPerSecondOperator !== '') {
                        params.stepPerSecondOperator = this.filters.stepPerSecondOperator
                    }
                    if (this.filters.stepPerSecond !== '') {
                        params.stepPerSecond = parseFloat(_this.filters.stepPerSecond);
                    }
                    if (this.filters.anotherStepPerSecond !== '') {
                        params.anotherStepPerSecond = parseFloat(_this.filters.anotherStepPerSecond);
                    }
                    if (this.filters.distancePerStepOperator !== '') {
                        params.distancePerStepOperator = this.filters.distancePerStepOperator
                    }
                    if (this.filters.distancePerStep !== '') {
                        params.distancePerStep = parseFloat(_this.filters.distancePerStep);
                    }
                    if (this.filters.anotherDistancePerStep !== '') {
                        params.anotherDistancePerStep = parseFloat(_this.filters.anotherDistancePerStep);
                }
                // let paramss = new URLSearchParams();
                // paramss.append('universityId',this.universityId);
                // paramss.append('studentName',this.filters.studentName);
                this.loadingShow = this.$loading({
                    lock: true,
                    text: '正在处理请稍后',
                    spinner: 'el-icon-loading',
                    background: 'rgba(0, 0, 0, 0.01)'
                });
                let url = resources.exportRecodeDate();
                this.$ajax.post(url,encodeURI(qs.stringify(params),"utf-8"))
                    .then(res => {
                        this.loadingShow.close();
                        location.href = resources.host + '/downloads/file?fileName=' + res.data;
                    })
            },
            searchRecords() {
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "universityId": this.universityId
                };
                params.selectAll = true;
                let _this = this;
                if (this.filters.studentName !== '') {
                    params.studentName = this.filters.studentName
                }
                if (this.filters.studentNo !== '') {           
                    params.selectAll = false;
                    if (this.isClick === true) {
                        params.selectAll = true;
                    }
                    params.studentNo = this.filters.studentNo
                }
                console.log(this.filters.timeRange[0])
                if (this.filters.timeRange[0] !== '' && this.filters.timeRange[0] !== null) {
                    params.startTime = this.filters.timeRange[0].getTime()
                }
                if (this.filters.timeRange[1] !== '' && this.filters.timeRange[1] !== null) {
                    params.endTime = this.filters.timeRange[1].getTime()
                }
                if (this.filters.runningSportId !== '') {
                    params.runningSportId = this.filters.runningSportId
                }
                // if (this.filters.isMan !== '') {
                //     params.isMan = this.filters.isMan
                // }
                if (this.filters.isValid !== '' && this.filters.isValid !== 'ALL') {
                    params.isValid = this.filters.isValid
                }
                if (this.filters.qualified !== '' && this.filters.qualified !== 'ALL') {
                    params.qualified = this.filters.qualified
                }
                if (this.filters.qualified !== '') {
                    params.qualified = this.filters.qualified
                }
                if (this.filters.speedOperator !== '') {
                    params.speedOperator = this.filters.speedOperator
                }
                if (this.filters.speed !== '') {
                    params.speed = parseFloat(_this.filters.speed);
                }
                if (this.filters.anotherSpeed !== '') {
                    params.anotherSpeed = parseFloat(_this.filters.anotherSpeed);
                }
                if (this.filters.stepPerSecondOperator !== '') {
                    params.stepPerSecondOperator = this.filters.stepPerSecondOperator
                }
                if (this.filters.stepPerSecond !== '') {
                    params.stepPerSecond = parseFloat(_this.filters.stepPerSecond);
                }
                if (this.filters.anotherStepPerSecond !== '') {
                    params.anotherStepPerSecond = parseFloat(_this.filters.anotherStepPerSecond);
                }
                if (this.filters.distancePerStepOperator !== '') {
                    params.distancePerStepOperator = this.filters.distancePerStepOperator
                }
                if (this.filters.distancePerStep !== '') {
                    params.distancePerStep = parseFloat(_this.filters.distancePerStep);
                }
                if (this.filters.anotherDistancePerStep !== '') {
                    params.anotherDistancePerStep = parseFloat(_this.filters.anotherDistancePerStep);
                }
                this.getData(params);
            },
            getData(params) {
                console.log(params);
                let _this = this;
                _this.studentList = [];
                _this.allRecords = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${recordsQuery}`,
                    variables: params
                })
                    .then(res => {
                        console.log(res);
                        _this.loading = false;
                        _this.dataCount = res.data.data.allRecords.dataCount;
                        _this.studentList = res.data.data.allRecords.data;
                        if (this.studentList.length > 0) {
                            if (this.filters.studentNo === this.studentList[0].studentNo) {
                                this.isShow = true;
                            } else {
                                this.isShow = false;
                            }
                        }
                        _this.allRecords = res.data.data.allRecords;
                        _this.studentList.forEach(item => {
                            item.startTime = new Date(item.startTime).toLocaleString().replace(/:\d{1,2}$/, ' ');
                            for (let i = 0; i < _this.options.project.length; i++) {
                                if (item.runningSportId === _this.options.project[i].id) {
                                    item.project = _this.options.project[i].name;
                                }
                            }
                        });
                    });
            },
            getPath(id) {
                let _this = this;
                this.pathShow = true;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${pathQuery}`,
                    variables: { "id": id }
                })
                .then(res => {
                    _this.pathData = [];
                    _this.pathDataOrigin = res.data.data.runningActivity.data;
                    _this.pathDataOrigin.forEach(data => {
                        _this.pathData.push({
                            'isNormal': data.isNormal,
                            'lnglat': [data.longitude, data.latitude]
                        });
                    });

                    if (_this.pathData.length > 0) {
                        _this.drawPath(_this.pathData);
                    } else {
                        document.getElementById('container').innerHTML = '';
                        _this.$message('没有查询到轨迹数据~');
                    }
                });
            },
            drawPath(data) {
                let _this = this;
                var path = data;
                var linArr = [];
                var color = '';
                var polyline = '';
                var map = new AMap.Map('container', {
                    resizeEnable: true,
                    center: data[0].lnglat,
                    zoom: 15
                });
                // 设置起点终点
                var start = new AMap.Marker({
                    icon: "http://webapi.amap.com/theme/v1.3/markers/n/mark_b.png",
                    position: path[0].lnglat
                });
                start.setTitle('起点');
                start.setMap(map);
                var end = new AMap.Marker({
                    icon: "http://webapi.amap.com/theme/v1.3/markers/n/mark_r.png",
                    position: path[path.length - 1].lnglat
                });
                end.setTitle('终点');
                end.setMap(map);

                // 2个点画一条线
                var line = [];
                for (var i = 0; i < path.length; i++) {
                    line = path.slice(i, i + 2);
                    if (line[1]) {
                        linArr = [line[0].lnglat, line[1].lnglat];
                    } else {
                        linArr = [line[0].lnglat];
                    }
                    color = (line[1] && line[1].isNormal == false) ? 'red' : 'green';
                    polyline = new AMap.Polyline({
                        path: linArr,          //设置线覆盖物路径
                        strokeColor: color, //线颜色
                        strokeOpacity: 1,       //线透明度
                        strokeWeight: 3,        //线宽
                        strokeStyle: "solid",   //线样式
                        strokeDasharray: [10, 5] //补充线样式
                    });
                    polyline.setMap(map);
                }
            }
        },
        mounted: function () {
            this.getSports();
            //this.searchRecords();
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
