<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 提示区域 -->
            <el-alert
                title="添加商品的信息"
                type="info"
                center
                show-icon
                :closable="false">
            </el-alert>
            <!-- 步骤条 -->
            <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center	>
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>
            <!-- tab区 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
                <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforTabLeave" @tab-click="tabClicked">
                    <el-tab-pane label="基本信息" name="0">
                        <el-form-item label="商品名称" prop="goods_name">
                            <el-input v-model="addForm.goods_name"></el-input>
                        </el-form-item>
                        <el-form-item label="商品价格" prop="goods_price">
                            <el-input v-model="addForm.goods_price" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品重量" prop="goods_weight">
                            <el-input v-model="addForm.goods_weight" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品数量" prop="goods_number">
                            <el-input v-model="addForm.goods_number" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品分类" prop="goods_cat">
                            <el-cascader
                                v-model="addForm.goods_cat"
                                :options="cateList"
                                :props="{ expandTrigger: 'hover',label:'cat_name',value:'cat_id',children:'children' }"
                                @change="handleChange"
                                clearable>
                            </el-cascader>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品参数" name="1">
                        <el-form-item v-for="item in manyTableData" :label="item.attr_name" :key="item.attr_id">
                            <!-- 复选框组 -->
                            <el-checkbox-group v-model="item.attr_vals" >
                                <el-checkbox v-for="(cb,i) in item.attr_vals" :key="i" :label="cb" border ></el-checkbox>
                            </el-checkbox-group>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品属性" name="2">商品属性</el-tab-pane>
                        <el-form-item :label="item.attr_name" :key="item.attr_id" v-for="item in onlyTableData">
                            <el-input v-model="item.attr_vals"></el-input>
                        </el-form-item>
                    <el-tab-pane label="商品图片" name="3">
                        <!-- aciton：图片上传地址 -->
                        <el-upload
                            :action="uploadUrl"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            list-type="picture"
                            :headers="headersObj"
                            :on-success="handleSuccess"
                            >
                            <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-tab-pane>
                    <el-tab-pane label="商品内容" name="4">商品内容</el-tab-pane>
                </el-tabs>
            </el-form>
        </el-card>
        <!-- 图片预览 -->
        <el-dialog
            title="图片预览"
            :visible.sync="preViewVisible"
            width="500px">~
            <img :src="previewPath" alt="" class="previewImg">
        </el-dialog>
    </div>
</template>


<script>
export default {
    data(){
        return {
            activeIndex:'0',//激活的步骤
            addForm:{
                goods_name:'',
                goods_price:0,
                goods_weight:0,
                goods_number:0,
                goods_cat:[],//商品分类
                pics:[],//图片路径数组
            },
            addFormRules:{
                goods_name:[
                    {required:true,message:'请输入商品名称',trigger:'blur'}
                ],
                goods_price:[
                    {required:true,message:'请输入商品价格',trigger:'blur'}
                ],
                goods_weight:[
                    {required:true,message:'请输入商品重量',trigger:'blur'}
                ],
                goods_number:[
                    {required:true,message:'请输入商品数量',trigger:'blur'}
                ],
                goods_cat:[
                    {required:true,message:'请选择商品分类',trigger:'blur'}
                ],
            },
            cateList:[],//商品分类列表
            manyTableData:[],//商品参数
            onlyTableData:[],//商品属性
            uploadUrl:"http://127.0.0.1:8888/api/private/v1/upload",//上传图片地址
            //对象请求头
            headersObj:{
                Authorization: window.sessionStorage.getItem('token'),
            },
            previewPath:'',
            preViewVisible:false,
        }
    },
    created(){
        this.getCateList();
    },
    computed:{
        cateId(){
            if(this.addForm.goods_cat.length===3){
                return this.addForm.goods_cat[2]
            }
            return null;
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
        //级联选择器，触发该函数
        handleChange(){
            if(this.addForm.goods_cat.length!==3){
                this.addForm.goods_cat=[];
            }
        },
        beforTabLeave(activeName, oldActiveName){
            if(oldActiveName==="0" && this.addForm.goods_cat.length!==3){
                this.$message.error("请先选择商品分类");
                return false;
            }
            return true;
        },
        //面板点击事件
        tabClicked(){
            //如果进入商品参数
            if(this.activeIndex==="1"){
                this.getParamsList();
            }else if(this.activeIndex==="2"){
                //进入商品属性
                this.getAttrList();
            }
        },
        //获取商品参数数据
        async getParamsList(){
            const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:"many"}});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`获取商品参数失败,${res.meta.msg}`);
            }else{
                res.data.forEach(item=>{
                    item.attr_vals = item.attr_vals?item.attr_vals.split(' '):[];
                });
                this.manyTableData = res.data;
                return;
            }
        },
        //获取商品属性数据
        async getAttrList(){
            const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:"only"}});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`获取商品属性失败,${res.meta.msg}`);
            }else{
                this.onlyTableData = res.data;
                return;
            }
        },
        //预览图片
        handlePreview(){

        },
        //删除图片
        handleRemove(){

        },
        //上传图片成功，触发函数
        handleSuccess(response, file, fileList){
            const picInfo = { pic:response.data.tmp_path};
            this.addForm.pics.push(picInfo);
        },
        //删除图片
        handleRemove(file, fileList){
            const filePath = file.response.data.tmp_path;
            const index = this.addForm.pics.findIndex(item=>item.pic===filePath);
            this.addForm.pics.splice(index,1);
        },
        //	点击文件列表中已上传的文件时的钩子
        handlePreview(file){
            console.log(file);
            this.previewPath=file.response.data.url;
            this.preViewVisible =true;
        }
    }
}
</script>

<style lang="less" scoped>
.el-checkbox{
    margin: 0 8px 0 0 !important;
}
.previewImg{
    width: 100%;
}

</style>