<template>
    <div class="page-container">
        <div class="main-panel">
            <el-tabs v-model="filters.versionName" @tab-click="searchRecords">
                <el-tab-pane label="Android" name="Android">
                    <span slot="label"><i class="fa fa-android" aria-hidden="true"></i>Android</span>
                </el-tab-pane>
                <el-tab-pane label="iOS" name="iOS">
                    <span slot="label"><i class="fa fa-apple" aria-hidden="true"></i>iOS</span>
                </el-tab-pane>
            </el-tabs>
            <el-col class="table-panel">
                <el-col :span="16">
                    <el-form :inline="true" :model="filters">
                        <el-form-item label="发布状态">
                            <el-select class="sm" v-model="filters.status" placeholder="发布状态">
                                <el-option label="全部" value="all"></el-option>
                                <el-option label="已发布" value="publish"></el-option>
                                <el-option label="未发布" value="unpublish"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="更新时间">
                            <el-date-picker v-model="filters.timeRange" type="datetimerange" placeholder="选择日期时间">
                            </el-date-picker>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="searchRecords">筛选</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
                <el-col :span="6">
                    <el-button @click="editVersion(filters.versionName, 'new')" style="float: right;margin: 7px 10px 0 0;" size="small" type="success">发布新版本</el-button>
                </el-col>
                <table class="table">
                    <tr>
                        <th width="10%">版本名称</th>
                        <th width="5%">版本号</th>
                        <th width="20%">更新日志</th>
                        <th>是否强制升级</th>
                        <th width="33%" v-if="filters.versionName === 'Android'">apk url</th>
                        <th width="10%">发布状态</th>
                        <th width="9%">更新时间</th>
                        <th width="13%">操作</th>
                    </tr>
                    <tr v-for="version in versionList">
                        <td>{{version.versionName}}</td>
                        <td>{{version.versionCode}}</td>
                        <td>{{version.changeLog}}</td>
                        <td>{{version.isForced ? '是':'否'}}</td>
                        <td v-if="filters.versionName === 'Android'">{{version.downloadUrl}}</td>
                        <td>
                            <span>{{version.isPublished ? '已发布' : '未发布'}}</span>
                        </td>
                        <td>{{version.updatedAt}}</td>
                        <td>
                            <div v-if="!version.isPublished">
                                <el-button type="text" @click="editVersion(filters.versionName, 'edit', version.id)">编辑</el-button>
                                <el-button type="text" @click="">删除</el-button>
                                <el-button type="text" @click="">发布</el-button>
                            </div>
                        </td>
                    </tr>
                </table>

                <div class="page">
                    <el-pagination @current-change="searchRecords" :current-page.sync="pageNumber" :page-size="10" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>
    </div>
</template>

<script>
    import resources from '../../resources'
    
    export default {
        data() {
            return {
                universityId: resources.universityId,
                pageSize: 10,
                pageNumber: 1,
                dataCount: 0,
                loading: false,
                filters: {
                    versionName: 'Android',
                    timeRange: [],
                    status: 'all'
                },
                versionList: []
            }
        },
        methods: {
            //获取列表
            searchRecords() {
                let _this = this;
                let filters = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "platformId": this.filters.versionName === 'Android' ? 0 : 1
                };
                this.getData(filters);
            },
            getData(filters) {
                let _this = this;
                let url = resources.versions();
                this.$ajax({
                    method: 'get',
                    url: url,
                    timeout: 10000,
                    params: filters//this is important !
                }).then( (res) => {
                    res.data.obj.forEach(item => {
                        item.updatedAt = new Date(item.updatedAt).toLocaleString().replace(/:\d{1,2}$/,' ');
                    });
                    _this.versionList = res.data.obj;
                })
            },
            editVersion(versionName, type, versionId) {
                console.log(versionName, type, versionId)
                this.$router.push({ path: `/version/${versionName}/${type}?versionId=${versionId}`});
            }
        },
        mounted: function () {
            this.searchRecords();
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
        .table {
            width: 90%;
            border-collapse: collapse;
            th {
                background-color: #f7f7f7;
            }
            td {
                padding: 5px 10px;
                text-align: left;
            }
            td,
            th {
                border: 1px solid #ebebeb;
                height: 33px;
            }
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
        .el- {
            width: 145px;
        }
        .el-date-editor {
            width: 280px;
        }
        .pointer {
            cursor: pointer;
            color: #29b6f6;
        }
    }
</style>