<template>

    <div>
        <ul>
            <li><i class="iconfont icon-icon tubiao"></i>精选</li>
            <li><i class="iconfont icon-icon tubiao"></i><router-link to = 'recommend'>推荐</router-link></li>
            <li><i class="iconfont icon-AIzhushou tubiao "></i>AI抖音</li>
            <li><i class="iconfont icon-wodeguanzhu tubiao"></i>关注</li>
            <li><i class="iconfont icon-pengyou tubiao"></i>朋友</li>
            <li><i class="iconfont icon-wode tubiao"></i>我的</li>
            <li><i class="iconfont icon-zhibo tubiao"></i>直播</li> 
            <li><i class="iconfont icon-fangyingting tubiao"></i>放映厅</li>
            <li><i class="iconfont icon-duanju tubiao"></i>短剧</li>
        </ul>
    </div>
</template>

<script setup>
</script>

<style scoped>
.tubiao {
    font-size: 15px;
    margin-right: 10px;
}
    div {
        width: 130px;
        text-align: center;
        font-size:16px;
        font-weight: 400;
        color:rgb(197, 197, 200);
    }
    div ul li {
        padding:10px 0;
        margin: 0 20px;
        text-align: center;
        
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
    ul li:nth-child(3),
    ul li:nth-child(8){
        padding-left: 13px;
    }
</style>