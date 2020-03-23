<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/students' }">学生管理</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/students' }">班级列表</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/classdetail/' + classId }">班级详情</el-breadcrumb-item>
            <el-breadcrumb-item>学生信息详情</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="main-panel">
            <el-col :span="24" class="info panel">
                <el-col :span="24" class="title">
                    个人信息
                </el-col>
                <el-col>
                    <!--<img src="" alt="">-->
                    <el-col :span="1">
                        <span class="avater"></span>
                    </el-col>
                    <el-col :span="22" :offset="1">
                        <el-col :span="24" class="title">
                            姓名：{{info.name}} 性别：{{info.isMan ? '男': '女'}} 籍贯：{{info.native}} 年龄：{{info.age}}
                        </el-col>
                        <el-col :span="24">
                            学校：{{info.school}} 学院：{{info.college}} 专业：{{info.major}} 年段：{{info.grade}}级 班级：{{info.className}} 学号：{{info.studentNo}}
                        </el-col>
                        <el-col :span="24">
                            身高：{{info.height}} 体重：{{info.weight}} 肺活量：{{info.vc}}
                        </el-col>
                        <el-col :span="24">
                            <el-button type="text" @click="editTarget">修改个人运动指标</el-button>
                        </el-col>
                    </el-col>
                </el-col>
            </el-col>
            <el-col :span="24" class="overview panel">
                <el-col :span="20" class="title">
                    个人运动概况
                </el-col>
                <el-col :span="1" :offset="1">
                    <span class="pointer" @click="toggleType('week')" :class="{'current': countType === 'week'}">本周</span>
                </el-col>
                <el-col :span="1">
                    <span class="pointer" @click="toggleType('month')" :class="{'current': countType === 'month'}">本月</span>
                </el-col>
                <el-col :span="1">
                    <span class="pointer" @click="toggleType('term')" :class="{'current': countType === 'term'}">本学期</span>
                </el-col>

                <el-col :span="5">
                    <span class="title">累计运动距离</span>
                    <span class="count">40km</span>
                </el-col>
                <el-col :span="5" :offset="1">
                    <span class="title">累计运动时长</span>
                    <span class="count duration">2000min</span>
                </el-col>
                <el-col :span="5" :offset="1">
                    <span class="title">累计达标次数</span>
                    <span class="count pass">2次</span>
                </el-col>
                <el-col :span="5" :offset="1">
                    <span class="title">累计未达标次数</span>
                    <span class="count unpass">1次</span>
                </el-col>

            </el-col>
            <el-col :span="24" class="detail panel">
                <el-col :span="24" class="title-top title">
                    运动明细
                </el-col>
                <el-col :span="24">
                    <el-col :span="5" class="sport-panel">
                        <div>
                            运动项目
                            <span>快跑</span>
                        </div>
                        <div>
                            本次运动距离（单位：米）
                            <span>40</span> 超过全校
                            <em>10%</em>的学生
                        </div>
                        <div>
                            本次运动时长（单位：分钟）
                            <span>120</span> 超过全校
                            <em>10%</em>的学生
                        </div>
                        <div>
                            当日达标情况
                            <span><em>达标！</em></span>
                        </div>
                    </el-col>
                    <el-col :span="19" class="graph-panel">
                        <el-col :span="18" class="title">
                            运动概况
                        </el-col>
                        <el-col :span="2">
                            <span class="pointer" @click="toggleGraphType('week')" :class="{'current': countGraphType === 'week'}">本周</span>
                        </el-col>
                        <el-col :span="2">
                            <span class="pointer" @click="toggleGraphType('month')" :class="{'current': countGraphType === 'month'}">本月</span>
                        </el-col>
                        <el-col :span="2">
                            <span class="pointer" @click="toggleGraphType('term')" :class="{'current': countGraphType === 'term'}">本学期</span>
                        </el-col>
                        <el-col :span="24" class="graph">
                            图表
                        </el-col>
                    </el-col>
                </el-col>
            </el-col>
            <!-- 修改个人运动指标弹窗 -->
            <el-dialog title="修改个人运动指标" :visible.sync="dialogFormVisible" size="tiny">
                <el-form :label-position="labelPosition" :model="formLabelAlign">
                    <el-form-item>
                        <span>陈粒  体测成绩：优</span>
                    </el-form-item>
                    <el-form-item label="修改每周运动次数">
                        <el-input v-model="formLabelAlign.times" placeholder="正常要求：每周3次"></el-input>
                    </el-form-item>
                    <el-form-item label="生效时间">
                        <el-date-picker style="width:100%" v-model="formLabelAlign.effectiveTime" type="datetimerange" placeholder="选择时间范围">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item label="修改理由">
                        <el-select style="width:100%" v-model="formLabelAlign.reason" placeholder="修改理由">
                            <el-option label="事假" value="thing"></el-option>
                            <el-option label="病假" value="sick"></el-option>
                            <el-option label="体质衰弱" value="weak"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="详细描述" prop="desc">
                        <el-input type="textarea" v-model="formLabelAlign.detail"></el-input>
                    </el-form-item>
                </el-form>

                <div slot="footer" class="dialog-footer">
                    <el-button @click="dialogFormVisible = false">取 消</el-button>
                    <el-button type="primary" @click="confirmEidt">确 定</el-button>
                </div>
            </el-dialog>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'
    
    const infoQuery = `query(
        $id: Long
        $universityId: Long
        $classId: Long
        ){
        student(id:$id) {
            name
            studentNo
            isMan
        }
        university(id:$universityId) {
            name
        }
        class(id:$classId) {
            name
            majorId
            grade
        }
    }`;

    const majorQuery = `query($majorId: Long){
        major(id:$majorId) {
            name
            collegeId
        }
    }`;

    const collegeQuery = `query($collegeId: Long){
        college(id:$collegeId) {
            name
        }
    }`;
    export default {
        data() {
            return {
                universityId: resources.universityId,
                classId: this.$route.params.class_id,
                id: this.$route.params.id,
                majorId: 0,
                collegeId: 0,
                labelPosition: 'top',
                countType: 'week',
                countGraphType: 'week',
                info: {
                    name: 'xx',
                    isMan: false,
                    native: '福建xx',
                    age: 24,
                    school: 'xx大学',
                    college: 'xx学院',
                    major: 'xx信息工程',
                    grade: '20xx',
                    className: '营销x班',
                    studentNo: '1207142111',
                    height: '174cm',
                    weight: '60kg',
                    vc: '2500ml'
                },
                dialogFormVisible: false,
                formLabelAlign: {
                    times: '',
                    effectiveTime: '',
                    reason: '',
                    detail: ''
                },
                formLabelWidth: '120px'
            }
        },
        methods: {
            getInfo() {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${infoQuery}`,
                    variables: {
                        "id": this.id,
                        "universityId": this.universityId,
                        "classId": this.classId
                    }
                })
                .then(res => {
                    _this.info.name = res.data.data.student.name;
                    _this.info.studentNo = res.data.data.student.studentNo;
                    _this.info.isMan = res.data.data.student.isMan;
                    _this.info.school = res.data.data.university.name;
                    _this.info.className = res.data.data.class.name;
                    _this.info.grade = res.data.data.class.grade;
                    _this.majorId = res.data.data.class.majorId;
                })
                .then(() => {
                    this.$ajax.post(`${resources.graphQlApi}`, {
                        'query': `${majorQuery}`,
                        variables: {
                            "majorId": _this.majorId
                        }
                    })
                    .then(res => {
                        _this.info.major = res.data.data.major.name;
                        _this.collegeId = res.data.data.major.collegeId;
                    })
                    .then(() => {
                        this.$ajax.post(`${resources.graphQlApi}`, {
                            'query': `${collegeQuery}`,
                            variables: {
                                "collegeId": _this.collegeId
                            }
                        })
                        .then(res => {
                            _this.info.college = res.data.data.college.name;
                        });
                    });
                });

            },
            toggleType(type) {
                console.log(type);
                this.countType = type;
            },
            toggleGraphType(type) {
                console.log(type);
                this.countGraphType = type;
            },
            editTarget() {
                console.log('修改指标');
                this.dialogFormVisible = true;
            },
            confirmEidt() {
                console.log('确认修改');
                this.dialogFormVisible = false
            }
        },
        mounted: function () {
            this.getInfo();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .pointer {
            cursor: pointer
        }
        a,
        .current {
            text-decoration: none;
            color: #29b6f6;
        }
        .panel {
            border: 1px solid #d4d4d4;
            border-radius: 4px;
            margin: 10px 0;
            min-height: 20px;
            padding: 5px 15px 15px;
        }
        .info {
            .avater {
                display: inline-block;
                width: 50px;
                height: 50px;
                background-color: #ccc;
                border-radius: 50%;
            }
        }
        .title {
            line-height: 2.5;
            font-weight: bold;
            font-size: 14px;
        }
        .overview {
            .count {
                font-size: 28px;
                border-radius: 4px;
                background-color: #29b6f6;
                color: #fff;
                text-align: center;
                font-weight: bold;
                width: 213px;
                height: 45px;
                line-height: 45px;
                display: inline-block;
            }
            .duration {
                background-color: #98d87d;
            }
            .pass {
                background-color: #98c0ff;
            }
            .unpass {
                background-color: #f27b71;
            }
        }
        .detail {
            padding: 5px 0 0;
            .title-top {
                padding-left: 15px;
                border-bottom: 1px solid #d4d4d4;
            }
        }
        .sport-panel {
            border-right: 1px solid #d4d4d4;
            padding: 5px 15px 15px;
            div {
                font-size: 12px;
                color: #b8b8b8;
                margin: 26px 0 0 10px;
                span {
                    font-size: 28px;
                    color: #6a6a6a;
                    display: block;
                }
                em {
                    color: #8fd8b3;
                    font-weight: bold;
                    font-style: normal;
                }
            }
        }
        .graph-panel {
            padding: 10px;
        }
        .graph {}
    }
</style>