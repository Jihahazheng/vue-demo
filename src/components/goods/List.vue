<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row :gutter="20" class="cat_opt">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" clearable @clear="getGoodsList" v-model="queryInfo.query" >
                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary">添加商品</el-button>
                </el-col>
            </el-row>

            <!-- 表格 -->
            <el-table :data="goodsList" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="商品名称" prop="goods_name" ></el-table-column>
                <el-table-column label="商品价格(元)" prop="goods_price" width="100px"></el-table-column>
                <el-table-column label="商品重量" prop="goods_weight" width="100px"></el-table-column>
                <el-table-column label="创建时间" prop="add_time" width="170px">
                    <template slot-scope="scope">
                        {{scope.row.add_time | dateFomat}}
                    </template>
                </el-table-column>
                <el-table-column
                    label="操作"
                    width="150px"
                >
                    <template slot-scope="scope">
                        <el-button type="primary" size="mini">编辑</el-button>
                        <el-button type="dange" size="mini">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>

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
    </div>
</template>

<script>
export default {
    data(){
        return {
            goodsList:[],//商品分类数据
            //查询条件
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:5,
            },
            total:0,//总数据条数
        }
    },
    created(){
        this.getGoodsList();
    },
    methods:{
        //获取对应的商品列表
        async getGoodsList(){
            const {data:res} = await this.$http.get('goods',{params:this.queryInfo});
            if(res.meta.status !== 200){
                //操作失败
                return this.$message.error(`查询商品失败,${res.meta.msg}`);
            }else{
                this.goodsList = res.data.goods;
                this.total = res.data.total;
                return;
            }
        },
        handleSizeChange(newPageSize){
            this.queryInfo.pagesize = newPageSize;
            this.getGoodsList();
        },
        handleCurrentChange(newPageNum){
            this.queryInfo.pagenum = newPageNum;
            this.getGoodsList();
        },
    }
}
</script>

<style lang="less" scope>

</style>