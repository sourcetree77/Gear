<template>
  <div class="collapseItem">
    <div class="title" @click="toggle">
      {{title}}
      <transition name="fade">
        <gear-icon class="right-icon" :class="{ active: isActive }" name="right"></gear-icon>
      </transition>
    </div>
    <gear-springs :visible="open" duration="500">
      <div class="item-content">
        <slot></slot>
      </div>
    </gear-springs>
  </div>
</template>

<script>
  import Icon from '../../base/icon/icon'
  import Springs from '../../action/springs/springs'
  export default {
    name: "GearCollapseItem",
    components: {
      'gear-icon': Icon,
      'gear-springs': Springs
    },
    props: {
      title: {
        // 必传且是字符串
        type: String,
        required: true
      },
      name: {
        type: String,
        required: true
      }
    },
    data () {
      return {
        open: false,
        isActive: false,
        visible: false
      }
    },
    inject: ['eventBus'],
    mounted () {
      if (this.eventBus) {
        this.eventBus.$on('update:selected', (names) => {
          if (names.indexOf(this.name) >=0) {
            this.open = true
            this.isActive = true
          } else {
            this.open = false
            this.isActive = false
          }
        })
      }
    },
    methods: {
      toggle () {
        if (this.open) {
          this.eventBus && this.eventBus.$emit('update:removeSelected', this.name)
        } else {
          this.eventBus && this.eventBus.$emit('update:addSelected', this.name)
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../style/var";
  $grey: grey;
  $border-radius: 4px;
  .collapseItem {
    > .title {
      position: relative;
      border: 1px solid $grey;
      margin-top: -1px;
      margin-left: -1px;
      margin-right: -1px;
      min-height: 32px;
      display: flex;
      align-items: center;
      padding: 0 8px;
      background: $collapse-title-bg;

      .right-icon {
        display: inline-block;
        position: absolute;
        right: 0.5em;
        transition: all .5s;
      }

      .active {
        transform: rotateZ(90deg);
      }
    }
    &:first-child {
      > .title {
        border-top-left-radius: $border-radius;
        border-top-right-radius: $border-radius;
      }
    }
    &:last-child {
      > .title:last-child {
        border-bottom-left-radius: $border-radius;
        border-bottom-right-radius: $border-radius;
        margin-bottom: -1px;
      }
    }

    .item-content {
      padding: 30px 8px;
    }
  }
</style>