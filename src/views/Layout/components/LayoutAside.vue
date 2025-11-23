<template>
    <link rel="stylesheet" href="//at.alicdn.com/t/c/font_5071343_exo7suyt999.css">

    <div>
        <ul>
            <li><i class="iconfont icon-icon"></i>精选</li>
            <li>推荐</li>
            <li>AI抖音</li>
            <li>关注</li>
            <li>朋友</li>
            <li>我的</li>
            <li>直播</li> 
            <li>放映厅</li>
            <li>短剧</li>
        </ul>
    </div>
</template>

<script setup>
</script>

<style scoped>
    div {
        width: 110px;
        text-align: center;
        font-size:10px;
        font-weight: 400;
        color:rgb(197, 197, 200);
    }
    div ul li {
        padding:8px 0;
        margin: 0 20px;
        
    }
    div ul li:hover {
        color:#fff;
        border-radius: 10px;
        background-color: rgb(45, 47, 56);
    }
    div ul li:nth-child(1) {
        margin-top:0
    }
    ul li:nth-child(3),
    ul li:nth-child(6){
        padding-bottom: 10px;
        border-bottom: 1px solid rgb(56, 56, 58);
    }
</style>