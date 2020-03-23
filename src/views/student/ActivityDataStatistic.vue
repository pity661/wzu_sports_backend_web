<template>
    <div class="page-container">
        <div class="main-panel">
            <el-col class="table-panel">
                <template v-if="type == 0">
                    <el-table :data="runningActivityStatisticList"> 
                        <!-- <el-table-column prop="" label="记录编号" > -->
                        <!-- <template scope="scope"> -->
                            <!-- {{(activityInfo.pageNum - 1) * 10 + scope.$index + 1}} -->
                        <!-- </template> -->
                         <el-table-column
                            type="index"
                            width="50">
                        </el-table-column>
                        <!-- </el-table-column> -->
                        <el-table-column prop="name" label="学生姓名">
                            <template scope="scope">
                                {{allActivityRecord.name}}
                            </template>
                        </el-table-column>
                        <el-table-column prop="studentNo" label="学生学号">
                            <template scope="scope">
                                {{allActivityRecord.studentNo}}
                            </template>
                        </el-table-column>
                        <el-table-column prop="activityId" label="运动编号">
                        </el-table-column>
                        <el-table-column prop="locationTotalCount" label="记录点总数">
                        </el-table-column>
                        <el-table-column prop="distancePerStepAgainst" label="违背步幅的点数">
                        </el-table-column>
                        <el-table-column prop="speedAgainst" label="违背速度的点数" v-if="1 == 1">
                        </el-table-column>
                        <el-table-column label="运动轨迹" width="120">
                        <template scope="scope">
                            <el-button type="text" @click="getPath(scope.row.activityId)">查看轨迹</el-button>
                        </template>
                        </el-table-column>
                    </el-table>
                </template>

                <template v-else>
                    <el-table :data="areaActivityStatisticList"> 
                        <el-table-column prop="" label="记录编号" >
                        <template scope="scope">
                            {{(activityInfo.pageNum - 1) * 10 + scope.$index + 1}}
                        </template>
                        </el-table-column>
                        <el-table-column prop="name" label="学生姓名">
                            <template scope="scope">
                                {{allActivityRecord.name}}
                            </template>
                        </el-table-column>
                        <el-table-column prop="studentNo" label="学生学号">
                            <template scope="scope">
                                {{allActivityRecord.studentNo}}
                            </template>
                        </el-table-column>
                        <el-table-column prop="activityType" label="运动方式" >
                            <template scope="scope">
                                区域运动
                            </template>
                        </el-table-column>
                        <el-table-column prop="location" label="运动地点" >
                            <template scope="scope">
                                {{scope.row.location.name}}
                            </template>
                        </el-table-column>
                        <el-table-column prop="activityId" label="运动编号">
                        </el-table-column>
                        <el-table-column prop="locationTotalCount" label="记录点总数">
                        </el-table-column>
                        <el-table-column prop="locationAgainst" label="离开区域的点数">
                        </el-table-column>
                    </el-table>
                </template> 

                <div class="page">
                    <el-pagination @current-change="searchActivityDataStatistic" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
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
    import resources from '../../resources'

    // 获取跑步运动运动点统计数据
    const runnningActivityStatisticQuery = `
    query(
        $studentId: Long
        $pageSize: Int
        $pageNumber: Int){
        allActivityRecord:student(id: $studentId){
            name
            studentNo
            activityInfo:runningActivityDataStatistic(
                    pageSize: $pageSize
                    pageNumber: $pageNumber){
                pageNum
                dataCount
                pageSize
                data{
                    studentId
                    activityId
                    locationTotalCount
                    distancePerStepAgainst
                    speedAgainst
                }
            }
        }
    }
    `
    // 获取区域运动运动点统计数据
    const areaActivityStatisticQuery = `
    query(
        $studentId: Long
        $pageSize: Int
        $pageNumber: Int
        ){
            allActivityRecord:student(id: $studentId){
                name
                studentNo
                activityInfo:areaActivityDataStatistic(
                pageSize: $pageSize
                pageNumber: $pageNumber){
                    pageNum
                    dataCount
                    pageSize
                    data{
                        studentId
                        activityId
                        locationTotalCount
                        locationAgainst
                        location{
                            id
                            name
                        }
                    }
                }
            }
        }
    `

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
                studentId: this.$route.params.student_id,
                type: this.$route.params.type,
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                loading: false,
                studentNo: '',
                studentName: '',
                options: {
                    project: null
                },
                filters: {
                    studentName: '',
                    studentNo: '',
                    activityId: '',
                },
                runningActivityStatisticList: [],
                areaActivityStatisticList: [],
                allActivityRecord: [],
                activityInfo: [],
                pathShow: false,
                pathDataOrigin: [],
                pathData: []
            }
        },
        methods: {
            searchActivityDataStatistic() {
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "studentId": this.studentId
                };
                let _this = this;
                this.getData(params);
            },
            getData(params) {
                let _this = this;
                this.runningActivityStatisticList = [];
                this.areaActivityStatisticList = [];
                if (_this.type == 0) {

                    _this.$ajax.post(`${resources.graphQlApi}`, {
                        'query': `${runnningActivityStatisticQuery}`,
                        variables: params
                    })
                    .then(res => {
                        console.log(res);
                        _this.loading = false;
                        _this.dataCount = res.data.data.allActivityRecord.activityInfo.dataCount;
                        _this.runningActivityStatisticList = res.data.data.allActivityRecord.activityInfo.data;
                        _this.allActivityRecord = res.data.data.allActivityRecord;
                        _this.activityInfo = res.data.data.allActivityRecord.activityInfo;
                    });
                } else if (_this.type == 1) {

                    _this.$ajax.post(`${resources.graphQlApi}`, {
                        'query': `${areaActivityStatisticQuery}`,
                        variables: params
                    })
                    .then(res => {
                        console.log(res);
                        _this.loading = false;
                        _this.dataCount = res.data.data.allActivityRecord.activityInfo.dataCount;
                        _this.areaActivityStatisticList = res.data.data.allActivityRecord.activityInfo.data;
                        _this.allActivityRecord = res.data.data.allActivityRecord;
                        _this.activityInfo = res.data.data.allActivityRecord.activityInfo;
                    });
                }
            },
            getPath(id) {
                let _this = this;
                this.pathShow = true;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${pathQuery}`,
                    variables: { "id": id }
                })
                .then(res => {
                    console.log(res);
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
            this.searchActivityDataStatistic()
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
