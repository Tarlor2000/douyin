<script setup>
import LayoutHeader from './components/LayoutHeader.vue'
import LayoutAside from './components/LayoutAside.vue'  
</script>

<template>
    <LayoutHeader />
    <div class="layout-body">
    <LayoutAside />
    <div class="layout-content">
        <router-view />
    </div>
    </div>
</template>

<style scoped>
.layout-body {
    display: flex;
}
.layout-content {
    flex: 1;
    padding: 16px;
}
</style>