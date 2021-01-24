<template>
  <div class="gear-nav" :class="{ vertical }">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: 'GearNav',
    // 依赖
    provide() {
      return {
        root: this,
        vertical: this.vertical   // 注入 vertical , 为了让 nav-sub 组件读取
      }
    },
    props: {
      selected: {
        type: String
      },
      vertical: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        items: [],
        namePath: []
      }
    },
    mounted() {
      this.updateChildren()
      this.listenToChildren()
    },
    updated() {
      this.updateChildren()
    },
    methods: {
      // 接收到注入传递过来的实例并 push 到 items 数组中
      addItem(vm) {
        this.items.push(vm)
      },
      updateChildren(){
        this.items.forEach(vm => {
          if (this.selected === vm.name) {
            vm.selected = true
          } else {
            vm.selected = false
          }
        })
      },
      listenToChildren() {
        this.items.forEach(vm => {
          vm.$on('update:selected', (name) => {
            this.$emit('update:selected', name)
          })
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../style/var";
  .gear-nav {
    display: flex;
    border-bottom: 1px solid $nav-bottom-line;
    cursor: pointer;
    user-select: none;
    &.vertical {
      flex-direction: column;
      width: 200px;
      border: 1px solid $nav-bottom-line;
    }
  }
</style>