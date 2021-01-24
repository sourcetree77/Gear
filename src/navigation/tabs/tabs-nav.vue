<template>
  <div class="tabs-nav" ref="nav">
    <slot></slot>
    <div class="line" ref="line"></div>
    <div class="actions-wrapper">
      <slot name="actions"></slot>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'GearTabsNav',
    inject: ['eventBus'],  // inject 注入
    mounted() {
      this.eventBus.$on('update:selected',(item, vm) => {
        let {width, height, top, left} = vm.$el.getBoundingClientRect()
        console.log(this.$refs.nav.getBoundingClientRect().x)
        // 修正外边框有 margin 时候 line 的显示 bug
        let x = this.$refs.nav.getBoundingClientRect().x
        console.log(width, height, top, left)
        this.$refs.line.style.width = `${width}px`
        this.$refs.line.style.left = `${left - x}px`
        console.log(this.$refs.line.style.left)
      })
    }
  }
</script>

<style lang="scss" scoped>
  $tab-height: 40px;
  $theme-color: #70b0f4;
  .tabs-nav {
    display: flex;
    height: $tab-height;
    justify-content: flex-start;
    position: relative;
    border-bottom: 1px solid #ddd;
    > .line {
      position: absolute;
      bottom: 0;
      border-bottom: 1px solid $theme-color;
      transition: all .5s;
    }
    > .actions-wrapper {
      margin-left: auto;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0 1em;
    }
  }
</style>