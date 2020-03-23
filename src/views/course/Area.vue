<template>
    <div class="page-container">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/courses' }">学科管理</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/outdoortarget/'+id }">设置运动指标</el-breadcrumb-item>
            <el-breadcrumb-item v-if="type === 'add'">新增锻炼区域</el-breadcrumb-item>
            <el-breadcrumb-item v-if="type === 'edit'">编辑锻炼区域</el-breadcrumb-item>
        </el-breadcrumb>
        <el-steps :space="300" :active="step" class="step">
            <el-step title="设置锻炼区域"></el-step>
            <el-step title="填写运动指标"></el-step>
        </el-steps>
        <div v-if="step === 1">
            <el-col :span="24" class="title">
                设置中心坐标
            </el-col>
            <div class="map-container">
                <div id="container" class="map" tabindex="0"></div>
                <div class='info-panel'>
                    <p class='title'>选择模式</p>
                    <div class="operate">
                        <input type='radio' name='mode' value='dragMap' checked>拖拽地图模式</input>
                        </br>
                        <input type='radio' name='mode' value='dragMarker'>拖拽Marker模式</input>
                        <br>
                        <button id='start'>开始选点</button>
                        <button id='stop'>关闭选点</button>
                    </div>
                    <p class='title'>选址结果</p>
                    <div class="info">
                        <div>经纬度:
                            <p>{{form.latitude}}, {{form.longitude}}</p>
                        </div>
                        <div>地址:
                            <p>{{form.addr}}</p>
                        </div>
                        <div>区域锻炼半径(m) :
                            <el-input placeholder="半径" v-model="form.radius"></el-input>
                        </div>
                    </div>
                </div>
            </div>
            <el-col :span="24" class="title">
                区域介绍
            </el-col>
            <el-col :span="12">
                <el-form ref="form" :model="form" label-width="80px">
                    <el-form-item label="名称">
                        <el-input v-model="form.name"></el-input>
                    </el-form-item>
                    <el-form-item label="简介">
                        <el-input type="textarea" placeholder="请输入简介" v-model="form.description"></el-input>
                    </el-form-item>
                    <!-- <el-form-item label="过期时间">
                        <el-col :span="11">
                            <el-date-picker type="date" placeholder="选择日期" v-model="form.overdue_time"></el-date-picker>
                        </el-col>
                    </el-form-item> -->
                </el-form>
            </el-col>
        </div>
        <div v-if="step === 2">
            <table class="table">
                <tr>
                    <th></th>
                    <th>时长（单位：min）</th>
                </tr>
                <tr>
                    <td>达标</td>
                    <td>
                        <el-input size="small" v-model="form.qualifiedCostTime"></el-input>
                    </td>
                </tr>
            </table>
        </div>
        <el-col :span="24">
            <el-button type="primary" @click="operate">{{step === 1 ? '下一步' : '保存'}}</el-button>
        </el-col>
    </div>
</template>
<script>
    import resources from '../../resources'

    export default {
        data() {
            return {
                id: this.$route.params.sport_id,
                area_id: this.$route.params.area_id,
                type: this.$route.query.type,
                universityId: resources.universityId,
                step: 1,
                form: {
                    name: '',
                    description: '',
                    radius: 0, // 半径
                    addr: '',
                    isEnabled: false,
                    latitude: '', // 经度
                    longitude: '',  // 纬度
                    qualifiedCostTime: '' // 达标时长
                }

            }
        },
        methods: {
            getArea() {
                let url = resources.fixLocationOutdoorSportPoints(this.area_id);
                let params = new URLSearchParams();
                this.$ajax.get(url, params)
                .then(res => {
                    this.form = res.data.obj;
                    this.form.qualifiedCostTime = res.data.obj.qualifiedCostTime / 60;
                });
            },
            operate() {
                if (this.step == 1) {
                    this.step++;
                } else {
                    let area = this.form;
                    let url = resources.fixLocationOutdoorSportPoints();
                    if (this.type === 'edit') {
                        url = resources.fixLocationOutdoorSportPoints(area.id);
                    }
                    let params = new URLSearchParams();
                    params.append('description', area.description);
                    params.append('latitude', area.latitude);
                    params.append('longitude', area.longitude);
                    params.append('radius', area.radius);
                    params.append('name', area.name);
                    params.append('isEnabled', area.isEnabled);
                    params.append('addr', area.addr);
                    params.append('qualifiedCostTime', area.qualifiedCostTime*60);
                    params.append('universityId', this.universityId);
                    this.$ajax.post(url, params)
                    .then(res => {
                        if (res.status === 200 || res.status === 201) {
                            this.$router.push({ path: '/outdoortarget/' + this.$route.params.sport_id });
                        }
                    });
                }
            }
        },
        mounted: function () {
            let _this = this;
            // 编辑状态下才会获取原本保存的信息
            if (this.type === 'edit') {
                this.getArea();
            }

            // 引入高德地图api
            AMapUI.loadUI(['misc/PositionPicker'], function (PositionPicker) {
                var map = new AMap.Map('container', {
                    zoom: 16,
                    scrollWheel: false
                })
                var positionPicker = new PositionPicker({
                    mode: 'dragMap',
                    map: map
                });

                positionPicker.on('success', function (positionResult) {
                    _this.form.latitude = positionResult.position.lat;
                    _this.form.longitude = positionResult.position.lng;
                    _this.form.addr = positionResult.address;
                });
                positionPicker.on('fail', function (positionResult) {
                    _this.form.latitude = '';
                    _this.form.longitude = '';
                    _this.form.addr = ' ';
                });
                var onModeChange = function (e) {
                    positionPicker.setMode(e.target.value)
                }
                var startButton = document.getElementById('start');
                var stopButton = document.getElementById('stop');
                var dragMapMode = document.getElementsByName('mode')[0];
                var dragMarkerMode = document.getElementsByName('mode')[1];
                AMap.event.addDomListener(startButton, 'click', function () {
                    positionPicker.start(map.getBounds().getSouthWest())
                })
                AMap.event.addDomListener(stopButton, 'click', function () {
                    positionPicker.stop();
                })
                AMap.event.addDomListener(dragMapMode, 'change', onModeChange)
                AMap.event.addDomListener(dragMarkerMode, 'change', onModeChange);
                positionPicker.start();
                map.panBy(0, 1);

                map.addControl(new AMap.ToolBar({
                    liteStyle: true
                }))
            });
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
        .step {
            text-align: center;
        }
        .table {
            width: 30%;
            border-collapse: collapse;
            text-align: center;
            margin-bottom: 20px;
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
        .map-container {
            .el-input {
                width: 100px;
            }
            .map {
                height: 390px;
                width: 650px;
                float: left;
                outline: none;
            }
            .operate {
                padding: 10px;
                line-height: 2;
            }
            .info-panel {
                padding-bottom: 10px;
                color: #444;
                background-color: #f8f8f8;
                width: 40%;
                float: left;
                height: 100%;
            }
            .info {
                padding: 10px;
                font-weight: bold;
                p {
                    font-weight: normal
                }
            }

            .title {
                margin: 0 0 10px 0;
                background-color: #dadada;
            }
            button {
                border: solid 1px;
                background-color: #dadafa;
            }

            .title {
                padding-left: 15px;
                font-size: 14px;
            }
        }
    }
</style>