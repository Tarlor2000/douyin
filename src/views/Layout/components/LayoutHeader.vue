<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
const searchKeywordlist = ref([])
const searchKeyword = ref('')
const flag = ref(false)

const change = () => {
    flag.value = !flag.value
}



const handleSearch = () => {
    if (searchKeyword.value.trim()) {
        // 避免重复添加相同的搜索关键词
        if (!searchKeywordlist.value.includes(searchKeyword.value)) {
            searchKeywordlist.value.push(searchKeyword.value)
        }
        console.log(searchKeyword.value)
        const searchKeywordlistStr = JSON.stringify(searchKeywordlist.value)
        localStorage.setItem("searchKeywordlist", searchKeywordlistStr)
        searchKeyword.value = ''
    }
}

const clearHistory = () => {
    searchKeywordlist.value = []
    localStorage.removeItem('searchKeywordlist');
}

const removeHistory = (idx) => {
    if (idx > -1) {
        searchKeywordlist.value.splice(idx, 1)
        localStorage.setItem('searchKeywordlist', JSON.stringify(searchKeywordlist.value))
    }
}

onMounted(() => {
    const cache = localStorage.getItem('searchKeywordlist')
    if (cache) {
        try {
            const parsed = JSON.parse(cache)
            if (Array.isArray(parsed)) {
                searchKeywordlist.value = parsed
            }
        } catch (err) {
            console.warn('Failed to parse search history cache', err)
        }
    }
})
</script>

<template>
    <link rel="stylesheet" href="//at.alicdn.com/t/c/font_5071343_5538xjyguml.css">

    <header >
        <router-link to="/"><i class="iconfont icon-douyin1" style="  margin-left: 10px;"></i><span>抖音</span></router-link>
        <div class="search-box" @mouseenter="change()" @mouseleave="change()">
            <input 
                type="text" 
                placeholder="搜索你感兴趣的内容" 
                v-model="searchKeyword"
                class="search-input"
                @keyup.enter="handleSearch"
            >
            <button class="search-btn" @click="handleSearch">
                <span><i class="iconfont icon-sousuo"></i>搜索</span>
            </button>

        </div>

        <div class = 'else'>
            <ul>
                <li><div><i class="iconfont icon--diamond-l"></i></div><div>充钻石</div></li>
                <li><div><i class="iconfont icon-xiazai"></i></div><div>客户端</div></li>
                <li><div><i class="iconfont icon-mofabang"></i></div><div>壁纸</div></li>
                <li><div><i class="iconfont icon-tongzhi"></i></div><div>通知</div></li>
                <li><div><i class="iconfont icon-sixin"></i></div><div>私信</div></li>
                <li><div><i class="iconfont icon-tianjia"></i></div><div>投稿</div></li>
            </ul>
            <div class="login"><i class="iconfont icon-yonghu" style="font-size:12px"></i>登录</div>
            
        </div>
    </header>
    <div class="search-message" v-if = "flag" @mouseenter="flag = true" @mouseleave="flag = false">
        <div class ="search-history-tt" style = "color: rgb(168, 168, 173)">
            <div style="font-weight:bold;">历史记录</div>
            <div class="search-history-clear" @click = " clearHistory"><i class="iconfont icon-shanchu" style="font-size: 8px;"></i>清除记录</div>
        </div>
        <div class = 'history'>
            <ul>
                <li 
                    v-for = "(item, index) in searchKeywordlist" 
                    :key = "item"
                    class="history-item"
                >
                    <span>{{item}}</span>
                    <button class="history-remove" @click.stop="removeHistory(index)">
                        ×
                    </button>
                </li>
            </ul>
        </div>
        <div class = "search-history-tt" style = "color: rgb(168, 168, 173) ;font-weight:bold;">猜你想搜</div>
        <div class = "search-history-tt" style = "color: rgb(168, 168, 173) ;font-weight:bold;">抖音热点</div>

    </div>
    
</template>

<style scoped>
    header {
        display: flex;
        justify-content: space-between;
        align-items: center;

    }
    .search-box {
        padding: 5px 0;
        display: flex;
        align-items: center;
        margin-left: 50px;
        
    }
    .search-input {
        width: 400px;
        padding: 8px 10px;
        border: 1px solid rgb(89, 90, 98);
        border-right: none;
        border-radius: 10px 0 0 10px;
        outline: none;
        font-size: 16px;
        background-color: rgb(57, 58, 68);
        color: #fff;
        transition: background-color 0.3s, border-color 0.3s;
    }
    .search-input:focus {
        border-color: #fff;
        background-color: rgb(22,24,35);
    }
    /* 当 input 获得 focus 时，button 也同步变化 */
    .search-input:focus ~ .search-btn {
        background-color: #fff;
        color: #000;
    }
    .search-btn {
        padding: 0 18px 0 0;
        height: 40px;
        border: 1px solid rgb(89, 90, 98);
        border-left: none;
        border-radius: 0 10px 10px 0;
        background-color: rgb(57, 58, 68);
        color: #fff;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: background-color 0.3s, color 0.3s;
        font-size: 13px;
        font-weight:bold;
        box-sizing: border-box;
    }
    /* 当 search-box 包含 focus 状态的 input 时，button 也同步变化 */
    .search-box:has(.search-input:focus) .search-btn {
        background-color: #fff;
        color: #000;
    }
    /* 当鼠标悬停在 search-box 上时，input 和 button 同步变化 */
    .search-box:hover .search-input {
        border-color: #fff;
        background-color: rgb(22,24,35);
    }
    .search-box:hover .search-btn {
        background-color: #fff;
        color: #000;

    }
    /* 当鼠标悬停在 input 上时，button 也变化 */
    .search-input:hover ~ .search-btn {
        background-color: #fff;
        color: #000;
    }
    /* 当鼠标悬停在 button 上时，input 也变化（使用 :has() 选择器） */
    .search-box:has(.search-btn:hover) .search-input {
        border-color: #fff;
        background-color: rgb(22,24,35);
    }
    
    .search-btn:hover {
        background-color: #fff;
        color: #000;
        border: 1px solid #fff;

    }
    .search-btn:active {
        background-color: rgb(15, 16, 25);
    }
     .search-btn span {
        border-left: 1px solid rgb(89, 90, 98);
     }
    .search-btn span i {
        font-size:12px
    }
    i {
        margin-left: 10px;
    }
    input {
        width: 230px;
        margin-left: 50px;
        border:1px solid #ccc;
        border-radius: 3px;
    }
   .search-message {
        position: fixed;
        z-index: 10;
        top:45px;
        left:465px;
        width: 465px;
        background-color: rgb(37, 38, 50);
        border: 1px solid rgb(89, 90, 98);
        border-radius: 6px;
        color: #fff;
        font-size: 12px;
    }
    .search-history-tt {
        display: flex;
        justify-content: space-between;
        padding: 10px;
    }
    .search-history-clear:hover {
        color: #fff;
    }
    .else {
        display:flex;
        justify-content:space-between;
        min-width: 150px;
        
    }
    .else ul{
        display: flex;
        align-items: center;
        justify-content: flex-end;
        gap: 8px;
        font-size:10px;
        color:rgb(162, 163, 167);
    }
    .else ul li{
        display:flex;
        flex-direction: column;
        align-items: center;
    }
    .else ul li div {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 30px;
    }
    .else ul li div i {
        margin-left: 0;
        font-size:14px
    }
    .login {
        display:flex;
        align-items: center;
        justify-content: center;
        gap: 2px;
        width: 70px;
        height: 40px;
        background-color: rgb(254, 44, 85);
        border-radius: 5px;
        color: #fff;
        margin-left: 10px;
        font-size:14px;
    }
    .login i {
        margin-left: 0;
       
    }
    .history ul  {
        display: flex;
        flex-wrap:wrap;
        margin-left: 10px;
    }

    .history ul li {
        position: relative;
        display: inline-flex;
        align-items: center;
        height: 20px;
        padding: 0 3px;
        background-color: rgb(51, 52, 63);
        margin: 0 5px 5px 0;
        border-radius: 4px;
    }
    .history ul li:hover {
        background-color: rgb(93, 95, 103);
    }
    .history-remove {
        position: absolute;
        top: -4px;
        right: -4px;
        width: 8px;
        height: 8px;
        border: none;
        border-radius: 50%;
        background: rgb(51, 52, 63);
        color: #fff;
        font-size: 10px;
        line-height: 12px;
        display: none;
        align-items: center;
        justify-content: center;
    }
    .history ul li:hover .history-remove {
        display: flex;
    }
</style>