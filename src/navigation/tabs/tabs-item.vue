<template>
  <div class="tabs-item" @click="handleClick" :class="classes" :data-name="name">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: 'GearTabsItem',
    inject: ['eventBus'],  // inject 注入
    data () {
      return {
        active: false
      }
    },
    props: {
      disabled: {
        type: Boolean,
        default: false,
      },
      name: {
        type: [String, Number],
        required: true,
      },
    },
    // 计算属性监听 active 改变 class
    computed: {
      classes () {
        return {
          active: this.active,
          disabled: this.disabled
        }
      }
    },
    created () {
      if (this.eventBus) {
        this.eventBus.$on('update:selected', (name) => {
          if (name === this.name) {
            this.active = true
          } else {
            this.active = false
          }
        })
      }
    },
    methods: {
      handleClick () {
        if (this.disabled) { return }
        this.eventBus.$emit('update:selected', this.name, this)
        this.$emit('click',this)
      }
    }
  }
</script>

<style lang="scss" scoped>
  $theme-color: #70b0f4;
  $disabled-color: #DDD;
  .tabs-item {
    flex-shrink: 0;
    padding: 0 2em;
    cursor: pointer;
    height: 100%;
    display: flex;
    align-items: center;
    &.active {
      color: $theme-color;
      fill: $theme-color;
    }
    &.disabled {
      color: $disabled-color;
      fill: $disabled-color;
      cursor: not-allowed;
    }
  }
</style>