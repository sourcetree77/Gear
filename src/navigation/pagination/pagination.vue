<template>
  <div class="gear-pagination" :class="{ hide: hideSinglePage === true && totalPage <= 1 }">
    <span class="gear-pagination-nav prev" :class="{ disabled: currentPage === 1}"
          @click="onClickPage(currentPage - 1)">
      <gear-icon class="icon" name="left"></gear-icon>
    </span>
    <span v-for="page in pages" class="gear-pagination-item"
          :class="{ active : page === currentPage, separator: page === '...' }"
          @click="onClickPage(page)">
      {{page}}
    </span>
    <span class="gear-pagination-nav next" :class="{ disabled: currentPage === totalPage }"
          @click="onClickPage(currentPage + 1)">
      <gear-icon class="icon" name="right"></gear-icon>
    </span>
  </div>
</template>

<script>
  import Icon from '../../base/icon/icon'
  export default {
    name: 'GearPagination',
    components: {
      'gear-icon': Icon
    },
    props: {
      totalPage: {
        type: Number,
        required: true
      },
      currentPage: {
        type: Number,
        required: true
      },
      hideSinglePage: {
        type: Boolean,
        default: true
      },
    },
    computed: {
      // 依赖 totalPage 和 currentPage
      pages() {
        // 总数大于 9
        if (this.totalPage > 9) {
          let array = [1, this.totalPage, this.currentPage, this.currentPage - 1, this.currentPage - 2, this.currentPage + 1, this.currentPage + 2];
          if (this.currentPage <= 5) {
            array = [1, 2, 3, 4, 5, 6, 7, this.currentPage + 1, this.currentPage + 2, this.totalPage];
          }
          if (this.currentPage >= this.totalPage - 4) {
            array = [1, this.totalPage, this.currentPage, this.totalPage - 1, this.totalPage - 2, this.totalPage - 3, this.totalPage - 4, this.totalPage - 5, this.totalPage - 6];
          }
          array = unique(array.sort((a, b) => a - b));
          let pages = array.reduce((prev, current, index, array) => {
            prev.push(current);
            let length = prev.length;
            // 前后大于 2 时, 添加 ...
            if (prev[length - 2] && current - prev[length - 2] > 2) {
              prev.splice(prev.length - 1, 0, '...');
            }
            return prev;
          }, []);
          return pages = pages.filter(n => (n >= 1 && n <= this.totalPage) || n === '...');
        } else {
          let array = [this.totalPage, this.totalPage - 1, this.totalPage - 2, this.totalPage - 3, this.totalPage - 4, this.totalPage - 5, this.totalPage - 6, this.totalPage - 7, this.totalPage - 8]
          let pages = unique(array.sort((a, b) => a - b));
          pages = pages.filter(n => (n >= 1 && n <= this.totalPage));
          return pages
        }
      }
    },
    methods: {
      onClickPage(page) {
        if(page === '...') {
          console.log('不是page')
        } else {
          if(page >=1 && page <= this.totalPage) {
            console.log(page)
            this.$emit('update:currentPage',page)
          }
        }
      }
    }
  }

  // 去重函数
  function unique (array) {
    // Set 方法去重 但兼容不好
    // return [...new Set(array)]

    // 前提是已知是数字的去重方法
    const object = {}
    array.map((number) => {
      object[number] = true
    })
    return Object.keys(object).map((s) => parseInt(s, 10))
  }
</script>

<style lang="scss" scoped>
  @import "../../style/var";
  .gear-pagination {
    &.hide {
      display: none;
    }
    &-item {
      border: 1px solid $pagination-item-border;
      border-radius: $border-radius;
      /*padding: 0 4px;*/
      display: inline-flex;
      justify-content: center;
      align-items: center;
      font-size: 14px;
      min-width: 30px;
      height: 30px;
      margin: 0 4px;
      cursor: pointer;
      user-select: none;
      vertical-align: middle;
      transition: all .3s;
      &.active, &:hover {
        transition: all .3s;
        border-color: $theme-color;
      }
      &.active {
        cursor: default;
      }
      &.separator {
        border: none;
        cursor: default;
      }
    }
    &-nav {
      display: inline-flex;
      justify-content: center;
      align-items: center;
      vertical-align: middle;
      cursor: pointer;
      &.disabled {
        svg {
          fill: lighten($grey, 20%);
        }
        cursor: not-allowed;
      }
    }
    .prev {
      margin-right: 8px;
    }
    .next {
      margin-left: 8px;
    }
  }
</style>