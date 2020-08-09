<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <el-row>
                <el-col :span="8">
                    <el-input placeholder="请输入内容"
                        v-model="queryInfo.query"
                        clearable
                        @clear="getOrderList"
                        >
                        <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
                    </el-input>
                </el-col>
                <!-- 订单列表数据 -->
                <el-table :data="orderList" border stripe>
                    <el-table-column type="index" label="#"></el-table-column>
                    <el-table-column label="订单编号" prop="order_number"></el-table-column>
                    <el-table-column label="订单价格" prop="order_price"></el-table-column>
                    <el-table-column label="是否付款" prop="pay_status">
                        <template slot-scope="scope">
                            <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
                            <el-tag type="danger" v-else>未付款</el-tag>

                        </template>
                    </el-table-column>
                    <el-table-column label="是否发货" prop="is_send"></el-table-column>
                    <el-table-column label="下单时间" prop="create_time">
                        <template slot-scope="scope">
                            {{scope.row.create_time | dateFormat}}
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" >
                        <template slot-scope="scope">
                            <el-button type="primary" size="mini" icon="el-icon-edit"></el-button>
                            <el-button type="success" size="mini" icon="el-icon-location"></el-button>
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
            </el-row>
        </el-card>
    </div>
</template>

<script>
export default {
    data(){
        return{
            //获取用户列表的参数对象
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:5,
            },
            orderList:[],
            total:0,
        }
    },
    created(){
        this.getOrderList();
    },
    methods:{
        async getOrderList(){
            const {data:res} =await this.$http.get('orders',{params:this.queryInfo});
            if(res.meta.status !== 200){
                return this.$message.error("获取订单列表失败");
            }
            this.orderList = res.data.goods;
            this.total = res.data.total;
        },
        // 监听pagesize改变
        handleSizeChange(newSize){
            // console.log(newSize);
            this.queryInfo.pagesize = newSize;
            this.getOrderList();
        },
        // 监听pageNum改变
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getOrderList();
        },
    }
}
</script>


<style lang="less" scoped>

</style>