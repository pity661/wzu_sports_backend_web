<template>
    <div class="page-container">
        <div class="overview-panel">
            <el-row>
                <el-col :span="4" class="subject">
                    <span>体育</span>
                </el-col>
                <el-col :span="15" class="overview" :offset="1">
                    <span class="title">任课教师概况</span>
                    <table width="100%">
                        <tr class="th">
                            <td>男性</td>
                            <td>女性</td>
                            <td>总数</td>
                        </tr>
                        <tr class="counts">
                            <td><span>{{maleTeachersCount}}人</span></td>
                            <td><span class="female">{{femaleTeachersCount}}人</span></td>
                            <td><span class="total">{{maleTeachersCount + femaleTeachersCount}}人</span></td>
                        </tr>
                    </table>
                </el-col>
                <el-col class="subject-detail">
                    <span @click="goCourses">查看学科详情 ></span>
                </el-col>
            </el-row>
        </div>
        <div class="main-panel">
            <el-col :span="24" class="title">
                任课教师
            </el-col>
            <el-col :span="20">
                <el-form :inline="true" :model="filters">
                    <el-form-item>
                        <el-input v-model="filters.name" placeholder="输入教师姓名"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-input v-model="filters.jobNo" placeholder="输入教师工号"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-select class="filter-isMan" v-model="filters.isMan" placeholder="性别">
                            <el-option label="男" value="true"></el-option>
                            <el-option label="女" value="false"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="search">筛选</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-col :span="4">
                <el-form>
                    <el-form-item>
                        <el-button type="primary" @click="batchAddTeachers">批量创建教师账号</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-table :data="teacherList" style="width: 100%" v-loading="loading" element-loading-text="玩命加载中">
                <el-table-column label="姓名" width="180">
                    <template scope="scope">
                        <el-icon name="name"></el-icon>
                        <span class="pointer" @click="goTeacherDetail(scope.row.id)">{{ scope.row.name }}</span>
                    </template>
                </el-table-column>
                <el-table-column prop="jobNo" label="工号" width="180">
                </el-table-column>
                <el-table-column prop="sex" label="性别">
                </el-table-column>
            </el-table>

            <div class="page">
                <el-pagination @current-change="search" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                    :total="dataCount">
                </el-pagination>
            </div>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'

    // 获取任课教师概览数据
    const teachersQuery = `
    query($id: Long){
        university(id: $id){
            teachersCount
            maleTeachersCount
            femaleTeachersCount
        }
    }`;

    // 筛选教师
    const teacherQuery = `
        query(
            $universityId: Long
            $name: String
            $jobNo: String
            $isMan: Boolean
            $pageNumber: Int
            $pageSize: Int
        ){
            allData:searchTeachers(
                universityId: $universityId
                name: $name
                jobNo: $jobNo
                isMan: $isMan
                pageNumber: $pageNumber
                pageSize: $pageSize
            ){
                pageNum
                pageSize
                dataCount
                data{
                    name
                    jobNo
                    isMan
                    id
                }
            }
        }
    `;
    export default {
        data() {
            return {
                universityId: resources.universityId,
                maleTeachersCount: 0,
                femaleTeachersCount: 0,
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                filters: {
                    name: '',
                    jobNo: '',
                    isMan: '',
                    id: ""
                },
                loading: false,
                teacherList: []
            }
        },
        methods: {
            //获取教师列表
            search() {
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "universityId": this.universityId
                };
                if (this.filters.name !== '') {
                    params.name = this.filters.name
                }
                if (this.filters.jobNo !== '') {
                    params.jobNo = this.filters.jobNo
                }
                if (this.filters.isMan !== '') {
                    params.isMan = this.filters.isMan
                }
                this.getData(params);
            },
            getData(params) {
                let _this = this;
                this.teacherList = [];
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${teacherQuery}`,
                    variables: params
                })
                .then(res => {
                    _this.loading = false;
                    _this.dataCount = res.data.data.allData.dataCount;
                    res.data.data.allData.data.forEach(item => {
                        let listItem = {
                            id: "",
                            name: "",
                            jobNo: "",
                            isMan: false
                        };
                        listItem.name = item.name;
                        listItem.jobNo = item.jobNo;
                        listItem.id = item.id;
                        listItem.sex = item.isMan ? '男' : '女';
                        _this.teacherList.push(listItem);
                    });
                });
            },
            batchAddTeachers() {
                this.$router.push({ path: '/addteacher' });
            },
            goCourses() {
                this.$router.push({ path: '/courses' });
            },
            goTeacherDetail(id) {
                this.$router.push({ path: '/teacherdetail/' + id });
            },
            getCounts() {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${teachersQuery}`,
                    variables: {
                        "id": this.universityId
                    }
                })
                .then(res => {
                    _this.femaleTeachersCount = res.data.data.university.femaleTeachersCount;
                    _this.maleTeachersCount = res.data.data.university.maleTeachersCount;
                });
            }
        },
        mounted: function () {
            let params = {
                "pageSize": 10,
                "pageNumber": 1,
                "universityId": this.universityId
            }
            this.getData(params);
            this.getCounts();
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        width: 1170px;
        .overview-panel {
            border: 1px solid #d4d4d4;
            border-radius: 4px;
            height: 115px;
            padding: 10px 0 10px 15px;
            .subject {
                font-size: 30px;
                background: #9ccc65;
                color: #fff;
                font-weight: bold;
                text-align: center;
                width: 115px;
                height: 115px;
                line-height: 4;
                border-radius: 8px;
            }
            .overview {
                .th {
                    color: #666;
                }
                .counts span {
                    font-size: 28px;
                    border-radius: 4px;
                    background-color: #29b6f6;
                    color: #fff;
                    text-align: center;
                    font-weight: bold;
                    width: 213px;
                    height: 45px;
                    display: inline-block;
                }
                span.female {
                    background-color: #ff5722;
                }
                span.total {
                    background-color: #ebebeb;
                    color: #666;
                }
            }
            .subject-detail {
                display: inline-block;
                width: 184px;
                height: 85px;
                color: #ffffff;
                background-color: #dedede;
                font-size: 20px;
                border-radius: 4px;
                line-height: 4;
                font-weight: bold;
                text-align: center;
                margin-left: 85px;
                margin-top: 15px;
                cursor: pointer;
            }
        }
        .main-panel {
            overflow: hidden;
            border: 1px solid #d4d4d4;
            border-radius: 4px;
            margin-top: 10px;
            padding: 0 10px 0px 15px;
        }
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .filter-isMan {
            width: 90px;
        }
        .page {
            text-align: center;
            margin: 10px;
        }
        .pointer {
            cursor: pointer;
        }
    }
</style>
