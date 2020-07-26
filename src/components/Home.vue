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
            <el-aside width="200px">
                <!-- 侧边栏菜单区 -->
                <el-menu
                    background-color="#333744"
                    text-color="#fff"
                    active-text-color="#ffd04b"
                >
                    <el-submenu index="1">
                        <template slot="title">
                            <i class="el-icon-location"></i>
                            <span>导航一</span>
                        </template>
                        <el-menu-item index="1-4-1">
                            <template slot="title">
                                <i class="el-icon-location"></i>
                                <span>子导航</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <!-- 右侧内容主题 -->
            <el-main>Main</el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
    methods:{
        logout(){
            window.sessionStorage.removeItem("token");
            this.$router.push("/login");
        }
    }
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

</style>