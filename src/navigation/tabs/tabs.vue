<template>
  <div class="tabs">
    <slot></slot>
  </div>
</template>

<script>
  import Vue from 'vue'
  export default {
    name: 'GearTabs',
    props: {
      selected: {
        type: String,
        required: true
      },
      direction: {
        type: String,
        default: 'Horizontal',
        validator (value) {
          return ['Horizontal','vertical'].indexOf(value) >= 0
        }
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
    methods: {
      checkChildren () {
        if (this.$children.length === 0) {
          // throw new Error('tabs 的子组件应该是 tabs-nav 和 tabs-content，但您没有写入子组件。')
          console &&  console.warn &&
          console.warn('tabs 的子组件应该是 tabs-nav 和 tabs-content，但您没有写入子组件。')
        }
      },
      selectTab () {
        // 找到 item
        this.$children.forEach((vm) => {
          if (vm.$options.name === 'GearTabsNav') {
            vm.$children.forEach((childVm) => {
              if (childVm.$options.name === 'GearTabsItem' && childVm.name === this.selected) {
                console.log(childVm.$el)
                this.eventBus.$emit('update:selected', this.selected, childVm)
                // 通知外部
                this.eventBus.$on('update:selected', (name) => {
                  this.$emit('update:selected', name)
                })
              }
            })
          }
        })
      }
    },
    mounted () {
      this.checkChildren()
      this.selectTab()
    }
  }
</script>

<style lang="scss" scoped>
  .tabs {

  }
</style>