<template>
  <div class="collapse">
    <slot></slot>
  </div>
</template>

<script>
  import Vue from 'vue'
  export default {
    name: "GearCollapse",
    props: {
      single: {
        type: Boolean,
        default: false
      },
      selected: {
        type: Array,
      }
    },
    data () {
      return {
        eventBus: new Vue()
      }
    },
    provide () {
      return {
        eventBus: this.eventBus
      }
    },
    mounted () {
      // 初始通知子组件
      this.eventBus.$emit('update:selected', this.selected)
      // 添加
      this.eventBus.$on('update:addSelected', (name) => {
        // 深拷贝一份 selected, 遵循 vue 单向数据流
        let selectedCopy = JSON.parse(JSON.stringify(this.selected))
        if (this.single) {
          selectedCopy = [name]
        } else {
          selectedCopy.push(name)
        }
        // 通知外部
        this.$emit('update:selected', selectedCopy)
        // 再次通知子组件
        this.eventBus.$emit('update:selected', selectedCopy)
      })
      // 移除
      this.eventBus.$on('update:removeSelected', (name) => {
        let selectedCopy = JSON.parse(JSON.stringify(this.selected))
        let index = selectedCopy.indexOf(name)
        selectedCopy.splice(index,1)
        this.$emit('update:selected', selectedCopy)
        this.eventBus.$emit('update:selected', selectedCopy)
      })
    }
  }
</script>

<style lang="scss" scoped>
  $grey: grey;
  $border-radius: 4px;
  .collapse {
    border: 1px solid $grey;
    border-radius: $border-radius;
  }
</style>