<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getBrands">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="名称" sortable>
            </el-table-column>
            <el-table-column prop="englishName" label="英文名" sortable>
            </el-table-column>
            <el-table-column prop="firstLetter" label="首字母" sortable>
            </el-table-column>
            <el-table-column prop="sortIndex" label="序号" sortable>
            </el-table-column>
            <el-table-column prop="logo" label="品牌logo"  sortable>
            </el-table-column>
            <el-table-column prop="productType.name" label="商品类型" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" sortable>
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length==0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="size" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <!--<el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form v-model="brand" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="用户姓名">
                    <el-input v-model="brand.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名">
                    <el-input v-model="brand.englishName"></el-input>
                </el-form-item>
                <el-form-item label="首字母">
                    <el-input v-model="brand.firstLetter"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="brand.description"></el-input>
                </el-form-item>
                <el-form-item label="商品类型">
                    <el-input v-model="brand.productTypeId"></el-input>
                </el-form-item>
                <el-form-item label="序号">
                    <el-input v-model="brand.sortIndex"></el-input>
                </el-form-item>
                <el-form-item label="品牌logo">
                    <el-input v-model="brand.logo"></el-input>
                </el-form-item>
                <el-form-item label="创建时间">
                    <el-input v-model="brand.createTime" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="修改时间">
                    <el-input v-model="brand.updateTime"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>-->

        <!--新增界面-->
        <el-dialog title="新增/编辑" :visible.sync="addFormVisible" width="30%">
            <el-form :model="brand" label-width="80px" row-style="height:10px">
                <el-form-item label="编号" v-show="false">
                    <el-input v-model="brand.id"></el-input>
                </el-form-item>
                <el-form-item label="用户姓名">
                    <el-input v-model="brand.name"></el-input>
                </el-form-item>
                <el-form-item label="英文名">
                    <el-input v-model="brand.englishName"></el-input>
                </el-form-item>
                <el-form-item label="首字母">
                    <el-input v-model="brand.firstLetter"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="brand.description"></el-input>
                </el-form-item>
                <el-form-item label="商品类型">
                    <el-select v-model="brand.productTypeId" placeholder="商品类型">
                        <el-option
                                v-for="item in productTypes"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                   <!-- <el-cascader
                            :options="productTypes"
                            :props="propdata"
                            expand-trigger="hover"
                            :show-all-levels="false"
                            v-model="selectedOptions"
                    change-on-select>
                    </el-cascader>-->
                </el-form-item>
                <el-form-item label="序号">
                    <el-input v-model="brand.sortIndex"></el-input>
                </el-form-item>
                <el-form-item label="品牌logo">
                    <el-input v-model="brand.logo"></el-input>
                </el-form-item>
                <el-form-item label="创建时间">
                    <el-input v-model="brand.createTime" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="修改时间">
                    <el-input v-model="brand.updateTime" :disabled="true"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    export default {
        data() {
            return {
                filters: {
                    name: ''
                },
                brands: [],
                total: 0,
                page: 1,
                size: 10,
                listLoading: false,
                sels: [],//列表选中列
                brand: {
                    id: "",
                    createTime: "",
                    updateTime: "",
                    name: "",
                    englishName: "",
                    firstLetter: "",
                    description: "",
                    productTypeId: "",
                    sortIndex: "",
                    logo: ""
                },
                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //类型
                productTypes:[],
                //多级连选择的数据
                // selectedOptions:[],
                propdata: {
                    value: 'id',
                    label: 'name'
                }
            }
        },
        methods: {
            //性别显示转换
            formatSex: function (row, column) {
                return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            //获取用户列表
            getBrands() {
                //转圈圈
                this.listLoading = true;
                this.$http.post("/product/brand/page",{
                    page:this.page,
                    size:this.size,
                    keyword:this.filters.keyword
                }).then((res)=>{
                    let data = res.data;//PageList
                    this.brands = data.rows;
                    this.total = data.total;
                    //转圈圈结束
                    this.listLoading = false;
                })
            },
            //删除单个
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    let para = row.id ;
                    this.$http.delete("/product/brand/"+para,{

                    }).then((res) => {
                        this.listLoading = false;
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        //这里是因为删除后加载的页面不对
                        this.page=1;
                        this.total=1;
                        this.getBrands();
                    });
                }).catch(() => {

                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.addFormVisible = true;
                this.brand = Object.assign({}, row);
                //将时间格式处理成Long类型
                var nowDate = new Date();
                var year= nowDate.getFullYear();
                var month = nowDate.getMonth()+1;
                var today = nowDate.getDate();
                if(month >= 1 && month <=9){
                    month = "0" + month;
                }
                if(today >= 1 && today <=9){
                    today = "0" + today;
                }
                var currentdate = year + "-" + month + "-" + today ;
                var currentDateLong = new Date(currentdate.replace(new RegExp("-","gm"),"/")).getTime();

                this.brand.updateTime=currentDateLong;
                //查询类型
                this.$http.get("/product/productType/list").then((res)=>{
                    this.productTypes = res.data;
                })
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                //将时间格式处理成Long类型
                var nowDate = new Date();
                var year= nowDate.getFullYear();
                var month = nowDate.getMonth()+1;
                var today = nowDate.getDate();

                if(month >= 1 && month <=9){
                    month = "0" + month;
                }
                if(today >= 1 && today <=9){
                    today = "0" + today;
                }
                var currentdate = year + "-" + month + "-" + today ;
                var currentDateLong = new Date(currentdate.replace(new RegExp("-","gm"),"/")).getTime();
                this.brand = {
                    createTime: currentDateLong,
                    updateTime: "",
                    name: "",
                    englishName: "",
                    firstLetter: "",
                    description: "",
                    productTypeId: "",
                    sortIndex: "",
                    logo: ""
                };
                //查询类型
                this.$http.get("/product/productType/list").then((res)=>{
                    this.productTypes = res.data;
                })
            },
            /*//编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
                            editUser(para).then((res) => {
                                this.editLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.editFormVisible = false;
                                this.getBrands();
                            });
                        });
                    }
                });
            },*/
            //新增和编辑
            addSubmit: function () {
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.addLoading = true;
                    this.$http.post("/product/brand",this.brand).then((res)=>{
                        this.addLoading = false;
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                type: 'success',
                                message: '提交成功!'
                            });
                            this.addFormVisible = false;
                            this.getBrands();
                        }else{
                            this.$message({
                                type: 'error',
                                message: '提交失败!'
                            });
                        }
                        /*this.addFormVisible = false;
                        this.getBrands();*/
                    })
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    this.$http.delete("/product/brand/deleteMany",{
                        data:ids
                    }).then((res)=>{
                        this.listLoading = false;
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    })
                    /*batchRemoveUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });*/
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getBrands();
        }
    }

</script>

<style scoped>

</style>