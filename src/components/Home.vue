<template>
    <el-container class="home-container">
        <!-- 头部 -->
        <el-header>
            <div class="header-div" >
                <img style="width: 50px; height: 50px" src="../assets/logo.png" alt="">
                <span style="margin-left:15px">电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <!-- 侧边栏 -->
            <el-aside :width="isCollapse?'64px':'200px'">
                <div class="toggle-button" @click="toggleCollapse">|||</div>
                <!-- 侧边栏菜单区 -->
                <el-menu
                    background-color="#333744"
                    text-color="#fff"
                    active-text-color="#409eff"
                    unique-opened
                    :collapse="isCollapse"
                    :collapse-transition= "false"
                    router
                    :default-active="activePath"
                >
                    <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
                        <template slot="title">
                            <i class="el-icon-menu"></i>
                            <span>{{item.authName}}</span>
                        </template>
                        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
                        @click="saveNavState('/'+subItem.path)">
                            <template slot="title">
                                <span>{{subItem.authName}}</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <!-- 右侧内容主题 -->
            <el-main>
                <!-- 路由占位符 -->
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
    data(){
        return {
            //左侧菜单数据
            menuList:[],
            //是否折叠菜单
            isCollapse:false,
            //被激活的链接地址
            activePath:'',
        }
    },
    //生命周期函数
    created(){
        this.getMenuList();
        this.activePath = window.sessionStorage.getItem("activePath");
    },
    methods:{
        logout(){
            window.sessionStorage.removeItem("token");
            this.$router.push("/login");
        },
        //获取菜单
        async getMenuList(){
            const {data:res} = await this.$http.get('menus');
            if(res.meta.status !==200){
                return this.$message.error(res.meta.msg);
            }
            this.menuList = res.data;
        },
        //点击按钮，切换菜单折叠展开
        toggleCollapse(){
            this.isCollapse = !this.isCollapse;
        },
        // 保存链接的激活状态
        saveNavState(activePath){
            window.sessionStorage.setItem("activePath",activePath);
            this.activePath = activePath;
        }
    },

}
</script>

<style scoped>
.home-container{
    height: 100%;
}
.el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
}
.header-div{
    display: flex;
    align-items: center;
}
.el-aside{
    background-color: #333744;
}
.el-main{
    background-color: #eaedf1;
}
.el-menu{
    border-right: none;
} 
.toggle-button{
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>