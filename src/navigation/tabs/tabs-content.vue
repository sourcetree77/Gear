<template>
  <div class="tabs-content">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: 'GearTabsContent',
    inject: ['eventBus'],  // inject 注入
    created() {

    }
  }
</script>

<style lang="scss" scoped>
  .tabs-content {

  }
</style>