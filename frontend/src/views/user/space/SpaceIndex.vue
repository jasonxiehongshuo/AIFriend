<script setup>
import {useRoute} from "vue-router";

const route = useRoute()//取当前url相关信息router是用来带开一个新页面或者后退
</script>

<template>
个人空间{{route.params.user_id}}
</template>

<style scoped>

</style>