<template>
    <div class="login_container">
        <div class="login_box"></div>
    </div>
</template>

<script>
export default {
    
}
</script>

// scoped 单页面生效，不影响其他页面
<style lang="less" scoped>
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}
.login_box{
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);//向x轴，y轴移动"自身长宽"的50%,从而达到居中的效果
}
</style>