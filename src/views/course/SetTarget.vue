<template>
    <div class="page-container">
        <div class="seting">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
                <el-breadcrumb-item>设置运动指标</el-breadcrumb-item>
            </el-breadcrumb>
            <el-col :span="24">
                <span class="title">{{sport_data.name}}</span>
                 <!-- <span class="space">
                    运动数据采集间隔(单位：s):
                    <el-input size="small" v-model="sport_data.acquisitionInterval"></el-input>
                </span>  -->
                <table class="table">
                    <tr>
                        <th></th>
                        <th v-for="th in sport.ths">{{th}}</th>
                    </tr>
                    <tr>
                        <td>达标</td>
                        <td><el-input size="small" v-model="sport_data.qualifiedDistance"></el-input></td>
                        <td><el-input size="small" v-model="sport_data.qualifiedCostTime"></el-input></td>
                        <td>{{sport_data.speed}}</td>
                    </tr>
                    <tr>
                        <td>超速</td>
                        <td>同上</td>
                        <td><el-input size="small" v-model="sport_data.minCostTime"></el-input></td>
                        <td>{{sport_data.speed}}</td>
                    </tr>
                </table>
            </el-col>
            <div class="tip">
                运动达标指标，将会作为学生主要的运动目标显示在前端。而超速指标，只有在学生以更短的时间，完成规定的运动距离，结束运动后，系统才会在App内提示学生，本次运动未达标。
            </div>
            <el-col :span="24">
                <el-button class="submit-btn" type="primary" @click="submit">保存</el-button>
            </el-col>
        </div>
    </div>
</template>
<script>
    import resources from '../../resources'
    
    const queryProject = `
    query ($id: Long) {
      runningSport(id: $id) {
        name
        qualifiedDistance
        qualifiedCostTime
        minCostTime
        acquisitionInterval
      }
    }`;
    export default {
        data() {
            return {
                id: this.$route.params.sport_id,
                sport: {
                    ths: ['距离（单位：m）', '时长（单位：min）', '速度（m/s）']
                },
                sport_data: {
                    name: '测试数据',
                    qualifiedDistance: 1000,
                    qualifiedCostTime: 3600,
                    minCostTime: 3600,
                    acquisitionInterval: 5
                }
            }
        },
        methods: {
            submit() {
                let _this = this;
                let id = this.id;
                // 普通的ajax接口
                // 使用 application/x-www-form-urlencoded 格式化 
                // 参考：http://blog.csdn.net/fantian001/article/details/70193938
                let url = resources.runningSportsUpdate(id);
                let params = new URLSearchParams();
                params.append('name', this.sport_data.name);
                params.append('qualifiedDistance', this.sport_data.qualifiedDistance);
                params.append('qualifiedCostTime', this.sport_data.qualifiedCostTime*60);
                params.append('minCostTime', this.sport_data.minCostTime*60);
                params.append('acquisitionInterval', this.sport_data.acquisitionInterval);

                this.$ajax.post(url, params)
                .then(res => {
                    _this.$router.push({ path: '/courses' });
                });
            },
            getSport(id) {
                let _this = this;
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${queryProject}`,
                    variables: {
                        "id": id
                    }
                    })
                    .then(res => {
                        _this.sport_data = {
                            name: res.data.data.runningSport.name,
                            qualifiedDistance: res.data.data.runningSport.qualifiedDistance,
                            qualifiedCostTime: (res.data.data.runningSport.qualifiedCostTime/60).toFixed(0),
                            minCostTime: (res.data.data.runningSport.minCostTime/60).toFixed(0),
                            speed: (res.data.data.runningSport.qualifiedDistance/res.data.data.runningSport.qualifiedCostTime).toFixed(1),
                            acquisitionInterval: res.data.data.runningSport.acquisitionInterval
                        }
                    })
                    .catch(error => {
                        console.log(error);
                    });
            }
        },
        mounted: function () {
            this.getSport(this.id);
        }
    }

</script>
<style lang="scss" scoped>
    .page-container {
        color: #666;
        .panel {
            margin-top: 20px;
            width: 300px;
        }
        .title {
            color: #666;
            line-height: 2.5;
            font-size: 16px;
            font-weight: bold;
        }
        .table {
            width: 60%;
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
        .tip{
            width: 700px;
            font-size: 12px;
        }
        .submit-btn {
            margin-top: 20px;
        }
        .space{
            display: block;
            width: 400px;
            margin-bottom: 10px;
            .el-input{
                width: 100px;
            }
        }
    }
</style>