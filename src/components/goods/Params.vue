<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-alert
                title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                :closable="false"
                show-icon>
            </el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                      <el-cascader
                        v-model="selectedCateKeys"
                        :options="cateList"
                        :props="{ expandTrigger: 'hover',label:'cat_name',value:'cat_id',children:'children' }"
                        @change="handleChange"
                        clearable>
                    </el-cascader>
                </el-col>
            </el-row>
            <!-- 标签页区域 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染tag标签 -->
                                <el-tag v-for="(item,i) in scope.row.attr_vals" 
                                :key="i" closable @close="handleTagClose(i,scope.row)">
                                    {{item}}
                                </el-tag>
                                <!-- 输入文本框 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>    
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" :disabled="isBtnDisabled" size="mini"  @click="addDialogVisible=true">添加属性</el-button>
                    <!-- 静态属性表格 -->
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染tag标签 -->
                                <el-tag v-for="(item,i) in scope.row.attr_vals" 
                                :key="i" closable @close="handleTagClose(i,scope.row)">
                                    {{item}}
                                </el-tag>
                                <!-- 输入文本框 -->
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                    >
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>    
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <!-- 添加参数对话框 -->
        <el-dialog
            :title="'添加'+titleText"
            :visible.sync="addDialogVisible"
            width="500px"
            @close="addDialogClosed">
            <!-- 添加参数表单 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 修改参数对话框 -->
        <el-dialog
            :title="'修改'+titleText"
            :visible.sync="editDialogVisible"
            width="500px"
            @close="editDialogClosed">
            <!-- 添加参数表单 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data(){
        return{
            cateList:[],//商品分类数据
            //选中的值
            selectedCateKeys:[],
            activeName:'many',//被激活标签名称
            onlyTableData:[],
            manyTableData:[],
            addDialogVisible:false,//是否显示添加对话框
            //添加表单数据
            addForm:{
                attr_name:"",
            },
            addFormRules:{
                attr_name:[
                    {required:true,message:"请输入参数名称",trigger:'blur'}
                ]
            },
            editDialogVisible:false,//是否显示修改对话框
            //修改表单数据
            editForm:{
                attr_name:"",
            },
            editFormRules:{
                attr_name:[
                    {required:true,message:"请输入参数名称",trigger:'blur'}
                ]
            },
            inputVisible:false,//控制按钮与文本框的显示
            inputValue:'',//文本框输入的内容
        }
    },
    created(){
        this.getCateList();
    },
    computed:{
        //如果按钮需要被禁用，则返回true,否则返回false
        isBtnDisabled(){
            return this.selectedCateKeys.length!==3;
        },
        //当前选中的三级分类的id
        cateId(){
            if(this.selectedCateKeys.length===3){
                return this.selectedCateKeys[2];
            }
            return null;
        },
        //动态计算标题文本
        titleText(){
            if(this.activeName==="many"){
                return "动态参数";
            }else{
                return "静态属性";
            }
        }
    },
    methods:{
        //获取分类数据
        async getCateList(){
            const {data:res} = await this.$http.get('categories',{params:{type:3}});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询商品失败,${res.meta.msg}`);
            }else{
                this.cateList = res.data;
                console.log(this.cateList);
                return;
            }
        },
        //级联选择框变化触发函数
        handleChange(){
            this.getParamsData();
        },
        //标签页点击事件
        handleTabClick(tab, event){
            this.getParamsData();
        },
        //获取表格数据
        async getParamsData(){
            //请求后台，获取面版数据
            if(this.cateId!=null){
                const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}});
                if(res.meta.status !== 200){
                    //操作失败
                    return this.$message.error(`查询属性失败,${res.meta.msg}`);
                }else{
                    res.data.forEach(item=>{
                        item.attr_vals = item.attr_vals?item.attr_vals.split(' '):[];
                        //控制文本框的显示与隐藏
                        item.inputVisible = false;
                        //文本框中输入的值
                        item.inputValue='';
                    })
                    if(this.activeName==="many"){
                        this.manyTableData = res.data;
                    }else{
                        this.onlyTableData = res.data;
                    }
                    return;
                }
            }else{
                this.manyTableData=[];
                this.onlyTableData=[];
                return;
            }
        },
        addDialogClosed(){
            this.$refs.addFormRef.resetFields();
        },
        //添加参数
        addParams(){
            this.$refs.addFormRef.validate(async (valid) => {
            if (valid) {
                const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name:this.addForm.attr_name,
                    attr_sel:this.activeName,
                });
                if(res.meta.status !== 201){
                    //操作失败
                    return this.$message.error(`添加失败,${res.meta.msg}`);
                }else{
                    this.getParamsData();
                    this.addDialogVisible = false;
                    return this.$message.success(`添加成功`);
                }
                
            } else {
                console.log('error submit!!');
                return false;
            }
        });
            
            
        },
        //打开修改对话框
        async showEditDialog(id){
            const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
                params:{attr_sel:this.activeName}
            });
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询参数失败,${res.meta.msg}`);
            }
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields();
        },
        //修改参数
        editParams(){
            this.$refs.editFormRef.validate(async (valid) => {
                if (valid) {
                    const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
                        attr_name:this.editForm.attr_name,
                        attr_sel:this.activeName,
                    });
                    if(res.meta.status !== 200){
                        //操作失败
                        return this.$message.error(`修改失败,${res.meta.msg}`);
                    }else{
                        this.getParamsData();
                        this.editDialogVisible = false;
                        return this.$message.success(`修改成功`);
                    }
                    
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });
        },
        //删除参数
        removeParams(id){
            this.$confirm('是否删除该数据?', '', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(async () => {
                const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`);
                if(res.meta.status !== 200){
                    //操作失败
                    return this.$message.error(`删除失败,${res.meta.msg}`);
                }
                this.getParamsData();
                this.$message({
                    type: 'success',
                    message: '删除成功!'
                });
            });
        },
        //确认输入内容后，文本框失去焦点，或按下enter键
        async handleInputConfirm(row){
            let inputValue = row.inputValue;
            if (inputValue.trim().length>0) {
                const newVals = row.attr_vals.join(' ')+' '+inputValue;
                //发起请求，保存到数据库
                const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
                    {
                        attr_name:row.attr_name,
                        attr_sel:row.attr_sel,
                        attr_vals:newVals,
                    }
                );
                if(res.meta.status !== 200){
                    //操作失败
                    return this.$message.error(`添加失败,${res.meta.msg}`);
                }
                this.$message({
                    type: 'success',
                    message: '添加成功!'
                });
                //添加到前端
                row.attr_vals.push(inputValue);
                
            }
            row.inputVisible = false;
            row.inputValue = '';
        },
        //显示文本框
        showInput(row){
            row.inputVisible = true;
            //让文本框自动获得焦点
            // nextTick方法作用：当页面上元素被重新渲染之后，执行回调函数代码
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
        },
        //删除标签
        async handleTagClose(i,row){
            console.log("1245");
            //删除一项
            row.attr_vals.splice(i,1);
            //发起请求，保存到数据库
            const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
                {
                    attr_name:row.attr_name,
                    attr_sel:row.attr_sel,
                    attr_vals:row.attr_vals.join(' '),
                }
            );
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`删除标签失败,${res.meta.msg}`);
            }
            this.$message({
                type: 'success',
                message: '删除标签成功!'
            });
        },
    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin: 15px 0;
}
.el-tag{
    margin: 10px;
}
.input-new-tag{
    width: 150px;
}
</style>