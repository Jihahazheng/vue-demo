<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="openAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 表格 -->
            <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" 
            :expand-type="false" show-index index-text="#" border>
                <!-- 是否有效 -->
                <template slot="isOk" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted=== false"
                    style="color: lightgreen;"></i>
                    <i class="el-icon-error" v-else style="color: red;"></i>
                </template>
                <!-- 排序 -->
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" type="success" v-if="scope.row.cat_level===0">一级</el-tag>
                    <el-tag size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
                    <el-tag size="mini" v-else-if="scope.row.cat_level===2" type="danger">三级</el-tag>
                </template>
                <!-- 操作 -->
                <template slot="opt" slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
                </template>
            </tree-table>

            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[5, 10, 25, 50]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加分类对话框 -->
        <el-dialog
            title="添加分类"
            :visible.sync="addCateVisible"
            width="500px"
            @close="addCateDialogClosed">
            <!-- 表单内容 -->
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef"
             label-width="100px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类" >
                      <el-cascader
                        v-model="selectedKeys"
                        :options="parentCateList"
                        :props="{ expandTrigger:'hover', label:'cat_name',value:'cat_id',children:'children' }"
                        @change="parentCateChanged"
                        clearable
                        change-on-select>
                    </el-cascader>
                </el-form-item>
            </el-form>

            <!-- 按钮区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return {
            catelist:[],//商品分类数据
            //查询条件
            queryInfo:{
                type:3,
                pagenum:1,
                pagesize:5,
            },
            total:0,//总数据条数
            //列表表头
            columns:[
                {
                    label:"分类名称",
                    prop:'cat_name',
                },
                {
                    label:"是否有效",
                    type:'template',//表示当前列定义为模板列
                    template:'isOk',//表示当前列使用的模板名称
                },
                {
                    label:"排序",
                    type:'template',//表示当前列定义为模板列
                    template:'order',//表示当前列使用的模板名称
                },
                {
                    label:"操作",
                    type:'template',//表示当前列定义为模板列
                    template:'opt',//表示当前列使用的模板名称
                },
            ],
            addCateVisible:false,//是否显示添加分类对话框
            //添加分类的表单数据对象
            addCateForm:{
                //分类名称
                cat_name:'',
                //父级分类id
                cat_pid:0,
                //分类等级，默认为一级分类
                cat_level:0,
            },
            //分类表单验证规则
            addCateFormRules:{
                cat_name:[
                    {required:true,message:'请输入分类名称',trigger:'blur'}
                ]
            },
            //父级分类数据
            parentCateList:[],
            //选中的值
            selectedKeys:[],
        }
    },
    created(){
        this.getCateList();
    },
    methods:{
        //获取商品分类
        async getCateList(){
            const {data:res} = await this.$http.get('categories',{params:this.queryInfo});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询商品失败,${res.meta.msg}`);
            }else{
                this.catelist = res.data.result;
                this.total = res.data.total;
                return;
            }
        },
        handleSizeChange(newPageSize){
            this.queryInfo.pagesize = newPageSize;
            this.getCateList();
        },
        handleCurrentChange(newPageNum){
            this.queryInfo.pagenum = newPageNum;
            this.getCateList();
        },
        openAddCateDialog(){
            //获取父级分类数据
            this.getParentCateList();
            this.addCateVisible=true;
        },
        //获取父级分类数据
        async getParentCateList(){
            const {data:res} = await this.$http.get('categories',{params:{type:2}});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询商品失败,${res.meta.msg}`);
            }else{
                this.parentCateList = res.data;
                console.log(this.parentCateList);
                return;
            }
        },
        //选择项发生时触发的函数
        parentCateChanged(){
            console.log(this.selectedKeys);
            if(this.selectedKeys.length>0){
               this.addCateForm.cat_pid= this.selectedKeys[this.selectedKeys.length-1];
               this.addCateForm.cat_level= this.selectedKeys.length;
            }else{
                this.addCateForm.cat_pid=0;
                this.addCateForm.cat_level=0;
            }
        },
        //添加新分类
        addCate(){
            this.$refs.addCateFormRef.validate(async (valid) => {
                if (valid) {
                    const {data:res} = await this.$http.post('categories',this.addCateForm);
                    if(res.meta.status !== 201){
                        //操作失败
                        return this.$message.error(`添加分类失败,${res.meta.msg}`);
                    }else{
                        this.getCateList();
                        this.addCateVisible=false;
                        return this.$message.success(`添加分类成功`);
                        
                    }  
                } else {
                    console.log('params error');
                    return false;
                }
            });
        },
        addCateDialogClosed(){
            this.$refs.addCateFormRef.resetFields();
            this.selectedKeys=[];
            this.addCateForm.cat_pid=0;
            this.addCateForm.cat_level=0;
        }
    }
}
</script>


<style lang="less" scoped>
.treeTable{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
}
</style>