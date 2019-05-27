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
        <el-dialog title="新增/编辑" :visible.sync="addFormVisible" width="30%" :before-close="closedialog">
            <el-form :model="brand" label-width="80px" row-style="height:10px" :rules="FormRules" ref="brand">
                <el-form-item label="编号" v-show="false">
                    <el-input v-model="brand.id"></el-input>
                </el-form-item>
                <el-form-item label="名称" prop="name">
                    <el-input v-model="brand.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名" prop="englishName">
                    <el-input v-model="brand.englishName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="序号" prop="sortIndex">
                    <el-input v-model="brand.sortIndex" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="logo">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/common/file/upload"
                            :before-remove="handleLogoRemove"
                            :file-list="logoList"
                            list-type="picture"
                            :limit="1"
                            :on-success="handleUploadSeccess"
                            :on-exceed="handleOutOfRange">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="商品类型">
                    <el-cascader style="width:100%"
                                 :options="productTypes"
                                  v-model="brand.productTypeId"
                                 :props="productProps"
                                 clearable change-on-select>
                    </el-cascader>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="brand.description" type="textarea" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="addFormVisible = false,closedialog()">取消</el-button>
                <el-button type="primary" @click="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    export default {

        data() {
            let englishName=(rule,value,callback)=>{
                let reg=/^[a-zA-Z]+$/
                if(!reg.test(value)){callback(new Error('只能输入英文字母!'))
                }else{
                    callback()
                }
            };
            let sortIndex=(rule,value,callback)=>{
                let reg=/^[0-9]*$/
                if(!reg.test(value)){callback(new Error('只能输入数字!'))
                }else{
                    callback()
                }
            };
            return {
                //级联选择器的属性配置
                productProps: {
                    label: "name",
                    value: "id"
                },
                //图片保存的东西，不知道是什么
                logoList:[],
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
                    name: "",
                    englishName: "",
                    description: "",
                    productTypeId: null,
                    sortIndex: "",
                    logo: ""
                },
                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                FormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ],
                    englishName: [
                        { required: true, message: '请输入英文名', trigger: 'blur' },
                        {validator:englishName,trigger:'blur'}
                    ],
                    sortIndex: [
                        { required: true, message: '请输入数字', trigger: 'blur' },
                        {validator:sortIndex,trigger:'blur'}
                    ],
                },
                //类型
                productTypes:[],
                //多级连选择的数据
                // selectedOptions:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                logodata:{
                    name:"",
                    url:""
                }
            }
        },
        methods: {
            //关闭模态框时清除验证,顺便在取消哪里
            closedialog(){
                this.$refs['brand'].resetFields();
                this.addFormVisible = false
            },
            //文件上传成功钩子函数
            handleUploadSeccess(response){
                this.brand.logo = response.data;
            },
            //文件超出个数
            handleOutOfRange(){
                this.$message({
                    message: '只能上传一张图片',
                    type: 'warning'
                });
            },
            //删除文件
            handleLogoRemove(file, fileList){
                //编辑时图片的删除
                if(!file.response){
                    this.logoList=[];
                }else{
                    this.$http.get("/common/file/delete",{
                        params:{
                            fileId:file.response.data
                        }
                    }).then(res=>{
                        let data = res.data;
                        if(data.success){
                            this.$message({
                                message: '删除成功',
                                type: 'success'
                            });
                        }else{
                            this.$message({
                                message: '删除失败',
                                type: 'error'
                            });
                            return false;//阻止删除
                        }
                    })
                }
            },
            //处理获取tree时的最后一个数据多了一个
            getTreeData(data) {
                // 循环遍历json数据
                for (var i = 0; i < data.length; i++) {
                    if (data[i].children.length < 1) {
                        // children若为空数组，则将children设为undefined
                        data[i].children = undefined;
                    } else {
                        // children若不为空数组，则继续 递归调用 本方法
                        this.getTreeData(data[i].children);
                    }
                }
                return data;
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
                })
            },
            searchtypes(){
                //查询类型
                this.$http.get("/product/productType/tree").then((res)=>{
                    this.productTypes = this.getTreeData(res.data);
                })
            },
            //回填时tree的处理
            changeDetSelect(key,treeData){
                let arr = []; // 在递归时操作的数组
                let returnArr = []; // 存放结果的数组
                let depth = 0; // 定义全局层级
                // 定义递归函数
                function childrenEach(childrenData, depthN) {
                    for (var j = 0; j < childrenData.length; j++) {
                        depth = depthN; // 将执行的层级赋值 到 全局层级
                        arr[depthN] = (childrenData[j].id);
                        if (childrenData[j].id == key) {
                            returnArr = arr.slice(0, depthN+1); //将目前匹配的数组，截断并保存到结果数组，
                            break
                        } else {
                            if (childrenData[j].children) {
                                depth ++;
                                childrenEach(childrenData[j].children, depth);
                            }
                        }
                    }
                    return returnArr;
                }
                this.brand.productTypeId = childrenEach(treeData, depth);
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.addFormVisible = true;
                this.brand = Object.assign({}, row);
                // 查询类型
                var data=this.productTypes
                var id=this.brand.productTypeId
                this.changeDetSelect(id,data);

                //要先清空，避免添加多个
                this.logoList=[];
                if(row.logo){
                    this.logodata.name=row.logo;
                    this.logodata.url="http://192.168.43.155"+row.logo;
                    this.logoList.push(this.logodata)
                }


            },
            //显示新增界面
            handleAdd: function () {
                this.logoList=[];
                this.addFormVisible = true;
                this.brand = {
                    name: "",
                    englishName: "",
                    description: "",
                    productTypeId: [],
                    sortIndex: "",
                    logo: ""
                }
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
                this.$refs.brand.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //格式化部分参数  productTypeId=[1,2,3]  ===> productTypeId=1
                            let para = Object.assign({}, this.brand);
                            para.productTypeId = para.productTypeId[para.productTypeId.length - 1];
                            this.$http.post("/product/brand",para).then((res)=>{
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
                    }
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
            this.searchtypes();
        }
    }

</script>

<style scoped>

</style>