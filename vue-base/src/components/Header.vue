<template>
    <div class="header-container">
    <img src="/images/todolist.png" alt="todolist" class="logo"></img>
    <h3 class="header">待办事项列表</h3>
    </div>
</template>

<script>
export default {

}
</script>

<style scoped>
.header-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 120px;
    gap: 20px;
}
.header {
    font-size: 50px;
    text-align: center;
    margin: 0;
}
.logo {
    width: 50px;
    height: 60px;
    border: 0;
}
</style>