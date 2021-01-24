<template>
  <div class="gear-nav-sub" :class="{ active, vertical }" v-click-outside="close">
    <span class="gear-nav-sub-label" @click="handleClick">
      <slot name="title"></slot>
      <span class="gear-nav-sub-icon" :class="{ open, vertical }">
        <gear-icon name="right"></gear-icon>
      </span>
    </span>
    <template v-if="vertical">
      <transition @enter="enter" @leave="leave" @after-leave="afterLeave"
                  @after-enter="afterEnter">
        <div class="gear-nav-sub-popover" v-show="open" :class="{ vertical }">
          <slot></slot>
        </div>
      </transition>
    </template>
    <template v-else>
      <div class="gear-nav-sub-popover" v-show="open">
        <slot></slot>
      </div>
    </template>
  </div>
</template>

<script>
  import Icon from '../../base/icon/icon'
  import ClickOutside from '../../plugins/click-outside'

  export default {
    name: 'GearNavSub',
    components: {
      'gear-icon': Icon
    },
    inject: ['root', 'vertical'],
    directives: {ClickOutside},
    props: {
      name: {
        type: String,
        required: true
      }
    },
    data() {
      return {
        open: false,
      }
    },
    computed: {
      active() {
        return this.root.namePath.indexOf(this.name) >= 0 ? true : false
      }
    },
    methods: {
      enter (el, done) {
        let {height} = el.getBoundingClientRect()
        el.style.height = 0
        el.getBoundingClientRect()
        el.style.height = `${height}px`
        el.addEventListener('transitionend', () => {
          done()
        })
      },
      afterEnter (el) {
        el.style.height = 'auto'
      },
      leave: function (el, done) {
        let {height} = el.getBoundingClientRect()
        el.style.height = `${height}px`
        el.getBoundingClientRect()
        el.style.height = 0
        el.addEventListener('transitionend', () => {
          done()
        })
      },
      afterLeave: function (el) {
        el.style.height = 'auto'
      },
      handleClick() {
        this.open = !this.open
      },
      close() {
        this.open = false
      },
      updateNamePath() {
        // this.active = true
        this.root.namePath.unshift(this.name)
        if (this.$parent.updateNamePath) {
          this.$parent.updateNamePath()
        } else {

        }
        // 选中 nav-item 关闭 popover
        // if(this.open = true) {
        //   this.close()
        // }
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../style/var";

  .gear-nav-sub {
    color: #999;
    transition: all .3s;
    position: relative;
    &:hover {
      color: #000;
      font-weight: bolder;
      transition: all .3s;
    }
    &.active {
      color: #000;
      font-weight: bolder;
    }
    &:not(.vertical) {
      &.active {
        color: #000;
        font-weight: bolder;
        &::after {
          content: '';
          position: absolute;
          bottom: 0;
          left: 0;
          border-bottom: 2px solid $theme-color;
          width: 100%;
        }
      }
      .gear-nav-sub-icon {
        display: none;
      }
    }
    &-icon {
      display: inline-flex;
      vertical-align: middle;
      margin-left: 1em;
      svg {
        fill: #999
      }
      transition: transform .3s;
      &.vertical {
        transform: rotate(90deg);
        &.open {
          transform: rotate(270deg);
        }
      }
    }
    &-label {
      padding: 10px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    &-popover {
      transition: height .3s;
      font-weight: normal;
      font-size: $font-size;
      background: white;
      position: absolute;
      z-index: 9;
      top: 100%;
      left: 0;
      margin-top: 1px;
      white-space: nowrap;
      box-shadow: 0 0 3px fade_out(black, .8);
      border-radius: $border-radius;
      min-width: 8em;
      &.vertical {
        position: static;
        border-radius: 0;
        border: none;
        box-shadow: none;
        overflow: hidden;
      }
    }
  }

  // 第二层级再往下 不采用一级样式
  .gear-nav-sub .gear-nav-sub {
    &.active {
      &::after {
        display: none;
      }
    }
    &.vertical {
      .gear-nav-sub-popover {
        padding-left: 6px;
        padding-right: 6px;
        margin-left: 0px;
      }
    }
    .gear-nav-sub-popover {
      top: 0;
      left: 100%;
      margin-left: 6px;
    }
    .gear-nav-sub-label {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .gear-nav-sub-icon {
      display: inline-flex;
      vertical-align: middle;
      margin-left: 1em;
      svg {
        fill: #999
      }
      transition: transform .3s;
      &.vertical {
        transform: rotate(90deg);
        &.open {
          transform: rotate(270deg);
        }
      }
      &.open {
        transform: rotate(180deg);
      }
    }
  }

</style>