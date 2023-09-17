<template>
  <header>
    <input class="header-search" type="text" v-model="searchName" placeholder="Search for games">
  </header>
</template>

<script>
</script>

<style lang="scss" scoped>
header{
  background: transparent;
}
.header-search{
  padding: 5px 10px;
  border: none;
  border-radius: 12px;
  background-color: #bcbcbc;
}
</style>
