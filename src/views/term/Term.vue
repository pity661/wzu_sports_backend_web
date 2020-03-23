<template>
    <div class="page-container">
        <div class="main-panel">
            <el-col class="table-panel">
                <el-form :inline="true">
                    <el-form-item>
                        <el-button type="primary" @click="addTerm()" class="add-new">新增</el-button>
                    </el-form-item>
                </el-form>

                <el-table :data="termList" style="width: 40%">
                    <el-table-column prop="" label="编号" width="80">
                        <template scope="scope">
                            {{(pageNumber - 1) * pageSize + scope.$index + 1}}
                        </template>
                    </el-table-column>
                    <el-table-column label="学期" width="230">
                        <template scope="scope">
                            <el-button type="text" @click="alterTerm(scope.row.id)">{{scope.row.name}}</el-button>
                        </template>
                    </el-table-column>
                    <el-table-column prop="showStartDate" label="开始时间" width="170">
                    </el-table-column>
                    <el-table-column prop="showEndDate" label="结束时间" width="170">
                    </el-table-column>
                </el-table>
                <div class="page" style="width: 40%">
                    <el-pagination @current-change="getTerm" :current-page.sync="pageNumber" :page-size="pageSize" layout="prev, pager, next, jumper"
                        :total="dataCount">
                    </el-pagination>
                </div>
            </el-col>
        </div>

        <el-dialog title="编辑学期" :visible.sync="dialog" size="mini">
            <el-form :model="dialogForm" :inline="true" label-width="100px">
                <el-form-item label="名称： ">
                    <el-input v-model="dialogForm.name" style="width:200px;" placeholder="学期名称">
                    </el-input>
                </el-form-item>
                <br/>
                <el-form-item label="开始时间：">
                    <div class="block">
                        <el-date-picker
                        v-model="dialogForm.startDate"
                        type="date"
                        placeholder="选择日期"
                        style="width:200px;">
                        </el-date-picker>
                    </div>
                </el-form-item>
                <br/>
                <el-form-item label="结束时间：">
                    <div class="block">
                        <el-date-picker
                        v-model="dialogForm.endDate"
                        type="date"
                        placeholder="选择日期"
                        style="width:200px;">
                        </el-date-picker>
                    </div>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialog = false">取 消</el-button>
                <el-button type="primary" @click="updateTerm()">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import resources from '../../resources'

    const termQuery = `
    query(
        $universityId: Long
        $pageNumber: Int
        $pageSize: Int
        ){
        pagingTerms(
            universityId:$universityId
            pageNumber:$pageNumber
            pageSize:$pageSize
            ){
            pagesCount
            dataCount
            pageNum
            data{
                id
                universityId
                name
                startDate
                endDate
            }
        }
    }
    `;
    
    export default {
        data() {
            return {
                universityId: resources.universityId,
                pageSize: 4,
                pageNumber: 1,
                dataCount: 0,
                loading: false,
                termList: [],
                dialog: false,
                dialogForm: {

                }
            }
        },
        methods: {
            addTerm() {
                let newObj = {
                    "name": '',
                    "startDate": '',
                    "endDate": ''
                };
                var { ...copyObj } = newObj;
                this.dialogForm = copyObj;
                this.dialog = true;
            },
            updateTerm() {
                if (this.dialogForm.startDate == '') {
                    this.lackMessage("开始时间不能为空")
                    return;
                }
                if (this.dialogForm.endDate == '') {
                    this.lackMessage("结束时间不能为空")
                    return;
                }
                if (this.dialogForm.name == '') {
                    this.lackMessage("学期名称不能为空")
                    return;
                }
                if (this.dialogForm.startDate > this.dialogForm.endDate) {
                    this.lackMessage("开始时间不能大于结束时间")
                    return;
                }
                let url = resources.terms(this.dialogForm.id);
                let params = new URLSearchParams();
                params.append('name',this.dialogForm.name);
                params.append('startDate',Number(this.dialogForm.startDate));
                params.append('endDate',Number(this.dialogForm.endDate));
                if (typeof (this.dialogForm.id) != 'undefined')  {
                    params.append('id',this.dialogForm.id);
                } else {
                    params.append('universityId',this.universityId);
                }
                this.$ajax.post(url,params)
                .then(res => {
                    console.log(res)
                    this.dialog = false;
                    this.succeed();
                    this.getTerm();
                })
            },
            alterTerm(id) {
                for (var i = 0; i < this.termList.length; i++) {
                    if (this.termList[i].id == id) {
                        //深拷贝
                       var { ...copyObj } = this.termList[i];
                       this.dialogForm = copyObj;
                    }
                }
                this.dialog = true;
            },
            getTerm() {
                let params = {
                    "pageSize": this.pageSize,
                    "pageNumber": this.pageNumber,
                    "universityId": this.universityId
                };
                this.$ajax.post(`${resources.graphQlApi}`, {
                    'query': `${termQuery}`,
                    variables: params
                }).then(res => {
                    console.log(res)
                    this.termList = res.data.data.pagingTerms.data;
                    this.dataCount = res.data.data.pagingTerms.dataCount;
                    this.termList.forEach(item => {
                        item.showStartDate = this.formateDate(item.startDate);
                        item.showEndDate = this.formateDate(item.endDate);
                    });
                })
            },
            formateDate(date) {
                var myDate = new Date(date);
                return myDate.getFullYear() + '年' + (myDate.getMonth() + 1) + '月' + myDate.getDate() + '日';
            },
            succeed(){
                this.$notify({
                    title: '成功',
                    message: this.result,
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
        },
        mounted: function () {
            this.getTerm();
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