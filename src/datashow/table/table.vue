<template>
  <div class="gear-table-wrapper" ref="tableWrapper">
    <div :style="{ height: height , overflow: 'auto' }" ref="tableWrapperOther">
      <table class="gear-table" :class="{ bordered, compact, striped }" ref="table">
        <thead>
        <tr>
          <th :style="{width: '40px'}">
            <input class="checkbox" type="checkbox" @change="onChangeAll" ref="allChecked" :checked="isAllSelected">
          </th>
          <th :style="{width: '50px'}" v-if="numberVisible">#</th>
          <th :style="{width: column.width + 'px'}" v-for="column in columns" :key="column.field">
            <div class="gear-table-header">
              {{column.text}}
              <span v-if="column.field in sortRules" class="gear-table-sorter" @click="changeSortRules(column.field)">
             <gear-icon name="sortup" :class="{ active: sortRules[column.field] === 'asc' }"></gear-icon>
             <gear-icon name="sortdown" :class="{ active: sortRules[column.field] === 'desc' }"></gear-icon>
            </span>
            </div>
          </th>
          <th ref="actionsHeader" v-if="$scopedSlots.default">
          </th>
        </tr>
        </thead>
        <tbody>
        <template v-for="item,index in dataSource">
          <tr :key="item.id">
            <td :style="{width: '40px'}">
              <input class="checkbox" type="checkbox" @change="onChangeCheckBox(item, index, $event)"
                     :checked="inSelectedItems(item)">
              <gear-icon class="gear-table-expend-icon" name="right"
                         v-if="dataSource[index].description" :class="{ open: isActive == item.id }"
                         @click="expendItem(item.id)"></gear-icon>
            </td>
            <td :style="{width: '50px'}" v-if="numberVisible">{{index + 1}}</td>
            <template v-for="column in columns">
              <td :style="{width: column.width + 'px'}" :key="column.field">{{item[column.field]}}</td>
            </template>
            <td v-if="$scopedSlots.default">
              <div ref="actions" style="display: inline-block;">
                <slot :item="item"></slot>
              </div>
            </td>
          </tr>
          <tr v-if="inExpendedId(item.id)" :key="`${item.id}-expend`">
            <td :style="{width: '20px', borderRight: '1px solid transparent'}"></td>
            <td :colspan="columns.length">
              {{item[expendField] || '空'}}
            </td>
          </tr>
        </template>
        </tbody>
      </table>
    </div>
    <div v-if="loading" class="gear-table-loading">
      <gear-icon name="loading"></gear-icon>
    </div>
  </div>
</template>

<script>
  import Icon from '../../base/icon/icon'
  import Springs from "../../action/springs/springs"
  export default {
    name: 'GearTable',
    components: {
      'gear-springs': Springs,
      'gear-icon': Icon
    },
    data(){
      return {
        isActive: false,
        // 判断 expendField 是否展开
        expendedId: []
      }
    },
    props: {
      selectedItems: {
        type: Array,
        default: () => []
      },
      columns: {
        type: Array,
        required: true
      },
      dataSource: {
        type: Array,
        required: true,
        // 保证每一项都有 id
        validator(array){
          if(array.filter(item => item.id === undefined).length > 0) {
            return false
          } else {
            return true
          }
        }
      },
      // 设置显示编号
      numberVisible: {
        type: Boolean,
        default: false
      },
      // 设置边框
      bordered: {
        type: Boolean,
        default: false
      },
      // 设置紧凑样式
      compact: {
        type: Boolean,
        default: false
      },
      // 奇偶行背景色
      striped: {
        type: Boolean,
        default: true
      },
      // 排序规则
      sortRules: {
        type: Object,
        default: () => ({}),
      },
      // 加载效果
      loading: {
        type: Boolean,
        default: false
      },
      height: {
        type: Number
      },
      // 展开行功能
      expendField: {
        type: String
      }
    },
    mounted() {
      let table2 = this.$refs.table.cloneNode(false)
      this.table2 = table2
      table2.classList.add('gear-table-copy')
      let thead = this.$refs.table.children[0]
      // 将 tr 整体下移一个 height 的高度
      let {height} = thead.getBoundingClientRect()
      // 将滚动条的高度下移一个 height 的高度
      this.$refs.tableWrapperOther.style.marginTop = height + 'px'
      this.$refs.tableWrapperOther.style.height = this.height - height + 'px'
      table2.appendChild(thead)
      this.$refs.tableWrapper.appendChild(table2)

      if(this.$scopedSlots.default) {
        let div = this.$refs.actions[0]
        let {width} = div.getBoundingClientRect()
        let parent = div.parentNode
        let styles = getComputedStyle(parent)
        let paddingLeft = styles.getPropertyValue('padding-left')
        let paddingRight = styles.getPropertyValue('padding-right')
        let borderLeft = styles.getPropertyValue('border-left-width')
        let borderRight = styles.getPropertyValue('border-left-width')
        console.log(paddingLeft,paddingRight,borderLeft,borderRight)
        let width2 = width + parseInt(paddingLeft) + parseInt(paddingRight) + parseInt(borderLeft) + parseInt(borderRight) + 'px'
        this.$refs.actionsHeader.style.width = width2
        // 对其所有 actions div
        this.$refs.actions.map(div => {
          div.parentNode.style.width = width2
        })
      }
    },
    beforeDestroy() {
      this.table2.remove()
    },
    watch: {
      selectedItems() {
        if(this.selectedItems.length === this.dataSource.length) {
          this.$refs.allChecked.indeterminate = false
        } else if(this.selectedItems.length === 0) {
          this.$refs.allChecked.indeterminate = false
          this.$refs.allChecked.checked = false
        } else {
          this.$refs.allChecked.indeterminate = true // 开启 checkbox 半选样式
        }
      }
    },
    computed: {
      // 所有都被选中
      isAllSelected(){
        // 当前显示数据的 id
        const a = this.dataSource.map(item => item.id).sort()
        // 用户选中数据的 id
        const b = this.selectedItems.map(item => item.id).sort()
        // 对比
        if(a.length === b.length) {
          let equal = true
          for(let i = 0; i < a.length; i++) {
            if(a[i] !== b[i]) {
              equal = false
              break
            }
          }
          return equal
        } else {
          return false
        }
      }
    },
    methods: {
      inSelectedItems(item){
        return this.selectedItems.filter((i) => i.id === item.id).length > 0
      },
      onChangeCheckBox(item, index, e) {
        let selected = e.target.checked
        let copy = JSON.parse(JSON.stringify(this.selectedItems))
        if(selected) {
          copy.push(item)
        } else {
          copy = copy.filter(i => i.id !== item.id)
        }
        this.$emit('update:selectedItems', copy)
      },
      onChangeAll(e) {
        let selected = e.target.checked
        if(selected) {
          this.$emit('update:selectedItems', this.dataSource)
        } else {
          this.$emit('update:selectedItems', [])
        }
      },
      // 排序
      changeSortRules(key) {
        const copy = JSON.parse(JSON.stringify(this.sortRules))
        let oldValue = copy[key]
        if(oldValue === 'asc') {
          copy[key] = 'desc'
        } else if (oldValue === 'desc') {
          copy[key] = true
        } else {
          copy[key] = 'asc'
        }
        this.$emit('update:sortRules', copy)
      },
      // 展开 expend
      expendItem(id) {
        if(this.inExpendedId(id)){
          this.expendedId.splice(this.expendedId.indexOf(id),1)
          this.isActive = false
        } else {
          this.expendedId.push(id)
          this.isActive = id
        }
      },
      // 检查是否展开
      inExpendedId(id) {
        return this.expendedId.indexOf(id) >= 0
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../style/var";
  $grey: lighten($grey,40%);
  .gear-table {
    width: 100%;
    border-collapse: collapse;   // 边框合并在一起
    border-spacing: 0;   // 边框间间距
    border-bottom: 1px solid $grey;
    &.bordered {
      border: 1px solid $grey;
      td,th {
        border: 1px solid $grey;
      }
    }
    &.compact {
      td,th {
        padding: 4px;
      }
    }
    &.striped {
      tbody {
        > tr {
          &:nth-child(odd) {
            background: #FFF;
          }
          &:nth-child(even) {
            background: $table-item-bg;
          }
        }
      }
    }
    td,th {
      border-bottom: 1px solid $grey;
      text-align: left;
      padding: 8px;
    }
    &-header {
      display: flex;
      align-items: center;
    }
    &-sorter {
      display: inline-flex;
      flex-direction: column;
      margin: 0 4px;
      svg {
        width: 8px;
        height: 8px;
        fill: $grey;
        cursor: pointer;
        &.active {
          fill: $theme-color;
        }
      }
    }
    &-wrapper {
      position: relative;
      overflow: auto;
    }
    &-loading {
      position: absolute;
      background: rgba(255,255,255,0.8);
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      svg {
        width: 50px;
        height: 50px;
        @include spin;
      }
    }
    &-copy {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      background: #FFF;
    }
    &-expend-icon {
      width: 14px;
      height: 14px;
      cursor: pointer;
      transition: all .3s;
      vertical-align: middle;
      position: relative;
      left: 7px;
    }
    .open {
      transform: rotateZ(90deg);
    }
    &-center {
      text-align: center !important;
    }

    .checkbox {
      vertical-align: middle;
    }
  }
</style>