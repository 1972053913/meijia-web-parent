<template>
        <el-container style="height: 630px">
            <el-aside width="300px">
                <el-tree :data="productTypes" :props="defaultProps" @node-contextmenu="handleNodeClick" @node-click="showneed"></el-tree>
            </el-aside>
            <el-main>
                <!--列表 不用table 只用来展示当前一个-->

                <!--新增界面-->
                <el-dialog title="新增/编辑" :visible.sync="addFormVisible" width="30%">
                    <el-form :model="productType" label-width="80px" row-style="height:10px">
                        <el-form-item label="编号" v-show="false">
                            <el-input v-model="productType.id"></el-input>
                        </el-form-item>
                        <el-form-item label="用户姓名">
                            <el-input v-model="productType.name"></el-input>
                        </el-form-item>
                        <el-form-item label="父级目录">
                            <!--<el-input v-model="productType.pid"></el-input>-->
                            <el-select v-model="productType.pid" placeholder="商品类型" :disabled="true">
                                <el-option
                                        v-for="item in productType1"
                                        :key="item.id"
                                        :label="item.name"
                                        :value="item.id">
                                </el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="路径">
                            <el-input v-model="productType.path"></el-input>
                        </el-form-item>
                        <el-form-item label="总数量">
                            <el-input v-model="productType.totalCount"></el-input>
                        </el-form-item>
                        <el-form-item label="标题">
                            <el-input v-model="productType.seoTile"></el-input>
                        </el-form-item>
                        <el-form-item label="关键字">
                            <el-input v-model="productType.seoKeywords"></el-input>
                        </el-form-item>
                        <el-form-item label="描述">
                            <el-input v-model="productType.description"></el-input>
                        </el-form-item>
                        <el-form-item label="商品类型">
                            <el-select v-model="productType.typeTemplateId" placeholder="商品类型">
                                <el-option
                                        v-for="item in productTypes"
                                        :key="item.id"
                                        :label="item.name"
                                        :value="item.id">
                                </el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="序号">
                            <el-input v-model="productType.sortIndex"></el-input>
                        </el-form-item>
                        <el-form-item label="品牌logo">
                            <el-input v-model="productType.logo"></el-input>
                        </el-form-item>
                    </el-form>
                    <div slot="footer" class="dialog-footer">
                        <el-button @click="addFormVisible = false">取消</el-button>
                        <el-button type="primary" @click="addSubmit" :loading="addLoading">提交</el-button>
                    </div>
                </el-dialog>
            </el-main>

            <div v-show="menuVisible">
                <ul id="menu" class="menu">
                    <li class="menu__item" @click="addCard">添加下级目录</li>
                    <li class="menu__item" @click="editCard">编辑该目录</li>
                    <li class="menu__item" @click="deleteCard">删除该目录及子目录</li>
                </ul>
            </div>
        </el-container>

</template>

<script>
    export default {
        data(){
            return {
                productTypes:[],
                productType:{
                    id:"",
                    name:"",
                    pid:"",
                    logo:"",
                    description:"",
                    sortIndex:"",
                    path:"",
                    totalCount:"",
                    seoTile:"",
                    seoKeywords:"",
                    typeTemplateId:""
                },
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                //这是对右键菜单的开关
                menuVisible:false,
                ids:"",
                //模态框属性
                addFormVisible:false,
                addLoading:false,
                productType1:"",//这是自关联查父级
                listLoading:false
            }
        },
        methods:{
            //当前选择的tree节点数据
            showneed(data,Node,value){
                //data表示的就是当前对象
                this.digui(data)//获取所有ids
                let id=(this.ids).slice(0,1).toString()//获取当前对象，是一个数字
                //通过id查询当前对象
                this.$http.get("/product/productType/"+id).then((res)=>{
                     console.debug(res)
                })
            },
            //加载树的数据
            loadTreeData(){
                this.$http.get("/product/productType/tree").then((res)=>{
                    this.productTypes = res.data;
                })
            },
            //使用递归查询所有的id值
            digui(value,arr=[]) {
                arr.push(value.id)
                //判断是否有子菜单
                var children = value.children;
                //有子菜单
                if(children.length){
                    for(var i =0;i<children.length;i++){
                        // arr.push(children[i].id)
                        //调用自己的方法，递归
                        this.digui(children[i],arr)
                    }
                }else{
                    //这里是没有子菜单
                    arr=new Array();
                    arr.push(value.id)
                }
                this.ids=arr;
            },
            //这是elementui自带的右键点击功能
            handleNodeClick(MouseEvent, object, Node, element){
                //object表示数据本身
                //调用方法获取ids
                this.digui(object);
                //////
                this.menuVisible = false // 先把模态框关死，目的是 第二次或者第n次右键鼠标的时候 它默认的是true
                this.menuVisible = true // 显示模态窗口，跳出自定义菜单栏
                var menu = document.querySelector('#menu')
                menu.style.left = MouseEvent.clientX - 160 + 'px'
                document.addEventListener('click', this.foo) // 给整个document添加监听鼠标事件，点击任何位置执行foo方法
                menu.style.top = MouseEvent.clientY - 10 + 'px'
            },
            foo() {
                // 取消鼠标监听事件 菜单栏
                this.menuVisible = false
                document.removeEventListener('click', this.foo) // 要及时关掉监听，不关掉的是一个坑，不信你试试，虽然前台显示的时候没有啥毛病，加一个alert你就知道了
            },
            //打开新增模态框
            addCard(){
                this.addFormVisible=true;
                this.$http.get("/product/productType/list", {
                }).then((res)=>{
                    this.productType1=res.data;
                })
                // console.debug(this.productType.pid)
            },
            //打开编辑模态框
            editCard(){
                this.addFormVisible=true;
            },
            //发送删除的请求
            deleteCard(){
                //将数组转换为string，并去掉【】号
                this.ids=JSON.stringify(this.ids).replace(/\[|]/g,'');
                this.$http.delete("/product/productType/deleteMany", {
                    data:this.ids
                }).then((res)=>{
                    this.$message({
                        message: '删除成功',
                        type: 'success'
                    });
                    this.loadTreeData();
                })
            },
            addSubmit(){

            }
        },
        mounted(){
            this.loadTreeData();
        }
    }
</script>

<style scoped>
    .el-aside{
        border:1px solid #ccc;
        border-right: none;
    }
    .el-main{
        border:1px solid #ccc;
    }
    .menu__item {
        display: block;
        line-height: 20px;
        text-align: center;
        margin-top: 10px;
    }
    .menu {
        height: 100px;
        width: 140px;
        position: absolute;
        border-radius: 10px;
        border: 1px solid #999999;
        background-color: #f4f4f4;
    }
    li:hover {
        background-color: #1790ff;
        color: white;
    }
</style>