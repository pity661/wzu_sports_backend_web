<template>
    <div class="page-container">
        <div class="seting">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
                <el-breadcrumb-item>设置运动指标</el-breadcrumb-item>
            </el-breadcrumb>
            <el-col :span="24">
                <el-col :span="19" class="title">
                    区域锻炼
                    <el-button class="operate-btn" @click="setArea" size="small" type="primary">新增锻炼区域</el-button>
                </el-col>
                <table class="table">
                    <tr>
                        <th>名称</th>
                        <th>地址</th>
                        <th width="200px">状态</th>
                        <th width="200px">操作</th>
                    </tr>
                    <tr v-for="area in area_data">
                        <td>{{area.name}}</td>
                        <td>{{area.addr}}</td>
                        <td>
                            <i class="dot" :class="{ 'dot-lock': !area.isEnabled }"></i> {{area.isEnabled ? '启用中' : '未启用'}}
                        </td>
                        <td>
                            <el-button v-if="area.isEnabled" type="text" @click="edit(area, false)">停用</el-button>
                            <div v-if="!area.isEnabled">
                                <el-button type="text" @click="edit(area, true)">启用</el-button>
                                <el-button type="text" @click="edit(area)">编辑</el-button>
                                <el-button type="text" @click="remove">删除</el-button>
                            </div>
                        </td>
                    </tr>
                </table>
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
                id: this.$route.params.sport_id,
                area_name: '区域锻炼',
                area_data: []
            }
        },
        methods: {
            getArea(id) {
                let url = resources.fixLocationOutdoorSportPoints();
                this.$ajax.get(url)
                .then(res => {
                    this.area_data = res.data.obj;
                });
            },
            setArea() {
                console.log('新增锻炼区域');
                this.$router.push({ path: `/area/${this.id}?type=add`});
            },
            remove() {
                console.log('remove，暂时还不做');
            },
            edit(area, isEnabled) {
                if (isEnabled === undefined) {
                    this.$router.push({ path: `/area/${this.id}/${area.id}?type=edit` });
                } else {
                    let url = resources.fixLocationOutdoorSportPoints(area.id);
                    let params = new URLSearchParams();
                    params.append('description', area.description);
                    params.append('latitude', area.latitude);
                    params.append('longitude', area.longitude);
                    params.append('radius', area.radius);
                    params.append('name', area.name);
                    params.append('isEnabled', isEnabled);
                    params.append('addr', area.addr);
                    params.append('qualifiedCostTime', area.qualifiedCostTime);
                    params.append('universityId', this.universityId);
                    this.$ajax.post(url, params)
                    .then(res => {
                        this.getArea(this.id);
                    });
                }
            }
        },
        mounted: function () {
            this.getArea(this.id);
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .table {
            width: 80%;
            border-collapse: collapse;
            text-align: center;
            th{
                background-color: #f7f7f7;
            }
            td{
                padding: 5px 20px;
            }
            td,th {
                border: 1px solid #ebebeb;
                height: 33px;
            }
        }
        .operate-btn {
            float: right
        }
        .dot {
            display: inline-block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #00a854;
        }
        .dot-lock {
            background: #bfbfbf;
        }
    }
</style>