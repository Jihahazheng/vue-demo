<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card class="box-card">
            <!-- 搜索与添加区域 -->
            <el-row :gutter='15'>
                <el-col :span="5">
                    <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getUserList"
                    >
                    <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            
            <!-- 用户列表区域 -->
            <el-table
                :data="userList"
                style="width: 100%"
                border
                stripe
            >
                <el-table-column
                    label="#"
                    type="index"
                >
                </el-table-column>
                <el-table-column
                    prop="username"
                    label="姓名"
                >
                </el-table-column>
                <el-table-column
                    prop="email"
                    label="邮箱"
                >
                </el-table-column>
                <el-table-column
                    prop="mobile"
                    label="电话"
                >
                </el-table-column>
                <el-table-column
                    prop="role_name"
                    label="角色"
                >
                </el-table-column>
                <el-table-column
                    prop="mg_state"
                    label="状态"
                >
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column
                    label="操作"
                >
                    <template slot-scope="scope">
                        <el-tooltip class="item" effect="dark" content="修改" placement="top" :enterable="false">
                            <el-button type="text" @click="showEditDialog(scope.row.id)" size="small">修改</el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" content="删除" placement="top" :enterable="false">
                            <el-button type="text" size="small">删除</el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" content="设置(分配角色)" placement="top" :enterable="false">
                            <el-button type="text" size="small">设置</el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
            >
            </el-pagination>

            <!-- 添加用户对话框 -->
            <el-dialog
                title="添加用户"
                :visible.sync="addDialogVisible"
                width="500px"
                @close="addDialogCloased"
                >
                <!-- 内容主体区 -->
                <el-form :model="addForm" status-icon :rules="addFormRules" ref="addFormRef"
                 label-width="70px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="addForm.username"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input type='password'  v-model="addForm.password"></el-input>
                    </el-form-item>
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="addForm.mobile"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="addForm.email"></el-input>
                    </el-form-item>
                </el-form>
                <!-- 底部按钮 -->
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 修改用户对话框 -->
            <el-dialog
                title="修改用户"
                :visible.sync="editDialogVisible"
                width="500px"
                @close="editDialogCloased"
                >
                <!-- 内容主体区 -->
                <el-form :model="editForm" status-icon :rules="editFormRules" ref="editFormRef"
                 label-width="70px">
                    <el-form-item label="用户名">
                        <el-input v-model="editForm.username" disabled></el-input>
                    </el-form-item>
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="editForm.mobile"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="editForm.email"></el-input>
                    </el-form-item>
                </el-form>
                <!-- 底部按钮 -->
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUser">确 定</el-button>
                </span>
            </el-dialog>

        </el-card>
    </div>
</template>

<script>
export default {
    data(){
        //验证邮箱的规则
        var checkEmail = (rule, value, callback) => {
            //正则
            const regEmail = /^[A-Za-z0-9_-]+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            //校验通过
            if(regEmail.test(value)){
                return callback();
            }
            callback(new Error('请输入合法邮箱'));
        }
        //验证手机号的规则
        var checkMobile = (rule, value, callback) => {
            //正则
            const regMobile = /^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            //校验通过
            if(regMobile.test(value)){
                return callback();
            }
            callback(new Error('请输入合法手机号'));
        }
        return {
            //获取用户列表的参数对象
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:2,
            },
            userList:[],
            total:0,
            addDialogVisible:false,//控制添加用户对话框的显示与隐藏
            //添加用户的表单数据
            addForm:{
                username:'',
                password:'',
                email:'',
                mobile:'',
            },
            //添加用户的表单校验规则
            addFormRules:{
                username:[
                    {required:true,message:'请输入用户名',trigger:'blur'},
                    {min:3,max:10,message:'用户名在3到10个字符间',trigger:'blur'}
                ],
                password:[
                    {required:true,message:'请输入密码',trigger:'blur'},
                    {min:6,max:15,message:'密码在3到10个字符间',trigger:'blur'}
                ],
                email:[
                    {required:true,message:'请输入邮箱',trigger:'blur'},
                    {validator: checkEmail, trigger: 'blur' }
                ],
                mobile:[
                    {required:true,message:'请输入手机号',trigger:'blur'},
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            editDialogVisible:false,//控制修改用户对话框的显示与隐藏
            editForm:{},
            editFormRules:{
                email:[
                    {required:true,message:'请输入邮箱',trigger:'blur'},
                    {validator: checkEmail, trigger: 'blur' }
                ],
                mobile:[
                    {required:true,message:'请输入手机号',trigger:'blur'},
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
        }
    },
    created(){
        this.getUserList();
    },
    methods: {
        async getUserList(){
            const {data:res} =await this.$http.get('users',{params:this.queryInfo});
            if(res.meta.status !== 200){
                return this.$message.error("获取用户列表失败");
            }
            this.userList = res.data.users;
            this.total = res.data.total;
        },
        // 监听pagesize改变
        handleSizeChange(newSize){
            // console.log(newSize);
            this.queryInfo.pagesize = newSize;
            this.getUserList();
        },
        // 监听pageNum改变
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getUserList();
        },
        //监听switch开关状态的改变
        async userStateChanged(userRecord){
            console.log(userRecord);
            const {data:res} = await this.$http.put(`users/${userRecord.id}/state/${userRecord.mg_state}`);
            if(res.meta.status !== 200){
                //操作失败，恢复状态
                userRecord.mg_state = !userRecord.mg_state;
                return this.$message.error("更新用户状态失败");
            }
            return this.$message.success("更新用户状态成功");
        },
        //监听添加用户对话框的关闭事件
        addDialogCloased(){
            this.$refs.addFormRef.resetFields();
        },
        //点击按钮，添加用户
        addUser(){
            this.$refs.addFormRef.validate(async valid =>{
                console.log(valid);
                if(!valid){
                    return;
                }
                const {data:res} = await this.$http.post("users",this.addForm);
                if(res.meta.status !== 201){
                    //操作失败
                    return this.$message.error(`添加用户失败,${res.meta.msg}`);
                }else{
                    this.$message.success("添加用户成功");
                    this.addDialogVisible = false;
                    //重新获取用户列表数据
                    this.getUserList();
                    return;
                }
                
            })
        },
        //展示编辑用户的对话框
        async showEditDialog(id){
            const {data:res} = await this.$http.get(`users/${id}`);
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询用户失败,${res.meta.msg}`);
            }else{
                this.editForm = res.data;
                this.editDialogVisible= true;
                return;
            }
            
        },
        //点击按钮，修改用户
        editUser(){
            this.$refs.editFormRef.validate(async valid =>{
                console.log(valid);
                if(!valid){
                    return;
                }
                const {data:res} = await this.$http.put(`users/${this.editForm.id}`,this.editForm);
                if(res.meta.status !== 200){
                    //操作失败
                    return this.$message.error(`修改用户失败,${res.meta.msg}`);
                }else{
                    this.$message.success("修改用户成功");
                    this.editDialogVisible = false;
                    //重新获取用户列表数据
                    this.getUserList();
                    return;
                }
                
            })
        },
        //监听修改用户对话框的关闭事件
        editDialogCloased(){
            this.$refs.editFormRef.resetFields();
        },
    }
}
</script>

<style lang="less" scoped>

</style>