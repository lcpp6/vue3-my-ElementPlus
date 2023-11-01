<template>
  <!--扩展分页功能 实现更多自定义扩展
  在父组件中使用示例：
  <MyPagination :currentPageNum="currentPageNum"
            :pageSize="pageSize"
            :total="total"
            :smallSize="true"
            :pageSizeOptions="[10,20,30,40,50]"
            @update-currentPageNum="(e)=>currentPageNum=e"
            @update-pageSize="(e)=>pageSize=e"/>
  -->
  <!--分页-->
  <div class="my-pagination">

    <!--每页 共多少条数据-->
    <div class="left-views" v-if="!showMoreLittle">
      <span class="text-total-left">共 {{ total }} 条</span>
      <!--输入设置页数-->
      <el-select v-model="pageSizeCopy" size="small" :disabled="disabledSelect"
                 :style="{width: getSelectAutoWidth+'px'}">
        <el-option v-for="item in pageSizeOptionsFilter" :key="item.value"
                   :label="item.label.length===1?('  '+item.label):(item.label)"
                   :value="item.value"/>
      </el-select>
      <span class="text-total-right">条/页</span>
    </div>
    <!--pager-count官网最低只能传5 但此处我尝试传3，以便达到更好的缩小效果（有警告/但能用/可忽略）:current-page="currentPageNumCopy"-->
    <el-pagination
        v-model="currentPageNumCopy"

        :page-size="pageSizeCopy"
        :total="total"
        :pager-count="smallSize?3:5"
        :hide-on-single-page="true"
        :small="smallSize"
        layout="prev, pager, next"
        @size-change="(e)=>pageSizeCopy=e"
        @current-change="currentChange">
    </el-pagination>
    <div class="right-views">
      <!--选择前往第几页  (e) => e>0?currentPageNumCopy=(e>total?total:e):currentPageNumCopy=1 -->
      <span class="text-page-left" v-if="!showMoreLittle">前往</span>
      <el-input-number v-model="inputNumber" :min="1" :max="maxPageNum" size="small" controls-position="right"
                       :style="{width: getInputAutoWidth+'px'}" @input="handleInput"
                       v-if="!showMoreLittle">
      </el-input-number>
      <!--提示：由于v-model会更改当前页面，因此必须使用新变量，并添加一个事件来辅助修改（输入值为空的时候是null）
      而且不能使用change事件change需要按下回车键才触发(开发时候发现的，也许是一个bug)...-->
      <el-input-number v-model="inputNumber" :min="1" :max="maxPageNum" :controls="false" style="margin: 0 20px 0 10px;"
                       :style="{width: getInputAutoWidth+'px'}"
                       size="small" @input="handleInput" v-else>
      </el-input-number>
      <span class="text-page-right" v-if="!showMoreLittle">页</span>
    </div>
  </div>
</template>

<script>

export default {
  name: "MyPagination",
  props: {
    currentPageNum: {// 当前页码
      default: 1
    },
    pageSize: {// 每页显示多少条数据
      default: 5
    },
    total: {// 总共有多少条/行数据
      default: 0
    },
    smallSize: {// 是否使用小尺寸的分页器(一般结合select使用)
      default: false
    },
    pageSizeOptions: {// 可供选择的页面大小：5行一页、10行一页...
      default() {
        return [5, 10, 20, 30, 50]
      }
    },

  },
  data() {
    return {
      currentPageNumCopy: 1,
      pageSizeCopy: 5,

      inputNumber: 1,
    };
  },
  // 计算属性
  computed: {
    maxPageNum() {
      const max = Math.ceil(this.total / this.pageSizeCopy)
      return max > 0 ? max : 1
    },
    disabledSelect() {
      return this.total < this.pageSizeCopy || (this.total < this.pageSizeCopy && this.maxPageNum === 1)
    },
    pageSizeOptionsFilter() {
      if (!this.pageSizeOptions) return []
      let pageSizeOptions = []

      for (let i = 0; i < this.pageSizeOptions.length; i++) {
        const item = this.pageSizeOptions[i];
        if (item > this.total) {
          if (this.maxPageNum === 1) {
            pageSizeOptions.push({
              value: 0,
              label: this.total.toString()
            })
          }
          break;// 如果当前值大于总数，就不再添加了
        }
        let obj = {
          value: this.pageSizeOptions[i],
          label: this.pageSizeOptions[i].toString()
        }
        pageSizeOptions.push(obj)
      }
      return pageSizeOptions
    },
    showMoreLittle() {// 是否显示更小的分页器
      return this.smallSize || Math.ceil(this.total / this.pageSizeCopy) <= 1
    },
    getSelectAutoWidth() {// 计算select的宽度
      return this.pageSizeOptionsFilter.length >= 1 ? (this.pageSizeOptionsFilter[this.pageSizeOptionsFilter.length - 1]).label.length * 8 + 35 : 8 + 35
    },
    getInputAutoWidth() {// 计算input的宽度
      return (this.currentPageNumCopy).toString().length * 8 + 35
    },

  },
  methods: {
    currentChange(e) {
      this.currentPageNumCopy = this.inputNumber = e
    },
    handleInput(e) {
      // (e) => e>0?currentPageNumCopy=(e>total/pageSizeCopy?total/pageSizeCopy:e):currentPageNumCopy=1
      console.log("handleInput = ", e)
      if (e === null || e <= 0) {
        this.inputNumber = 1;
        this.currentPageNumCopy = 1
      } else {
        const totalPageNum = Math.ceil(this.total / this.pageSizeCopy)// 向上取整
        const inputNumber = e > totalPageNum ? totalPageNum : e
        // console.log("handleInput = ", e, inputNumber)
        this.inputNumber = this.currentPageNumCopy = inputNumber
      }

    }
  },
  watch: {
    currentPageNum: {// 随着父组件的currentPageNum变化而变化
      handler(newVal) {
        this.currentPageNumCopy = newVal
      },
      immediate: true,// 初始化后立即触发一次
    },
    pageSize: {// 随着父组件的pageSize变化而变化
      handler(newVal) {
        // console.log("pageSize = ", newVal)
        this.pageSizeCopy = newVal
      },
      immediate: true,// 初始化后立即触发一次
    },
    currentPageNumCopy(newVal, oldVal) {
      // console.log("当前页码 发生变化了...", newVal, oldVal)
      if (newVal !== oldVal) {// 告诉父组件 我改了currentPageNum----父组件需要使用它来更新 当前页的数据
        this.$emit('updateCurrentPageNum', newVal)
      }
    },
    pageSizeCopy(newVal, oldVal) {
      if (newVal !== oldVal) {// 告诉父组件 我改了pageSize----父组件需要使用它来更新 每一页的数据量大小
        this.$emit('updatePageSize', newVal)
      }
    },
  }
  ,
};
</script>

<style scoped>
.my-pagination {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  align-items: flex-end;
  flex-wrap: nowrap;
  background-color: #ffffff;
  width: 100%;
  height: 30px;
}

@mixin text-mixin {
  font-size: small;
  display: flex;
  flex-direction: row;
  user-select: none;
}
.text-total-left {
  @include text-mixin;
  margin: 0 10px 0 5px;
}

.text-total-right {
  @include text-mixin;
  margin: 0 5px 0 10px;
}

.text-page-left {
  @include text-mixin;
  margin: 0 10px 0 5px;
}

.text-page-right {
  @include text-mixin;
  margin: 0 5px 0 10px;
}
</style>
