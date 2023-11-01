<template>
  <!--扩展分页功能 实现更多自定义扩展
  在父组件中使用示例：
  <MyPagination :currentPageNum="currentPageNum"
            :pageSize="pageSize"
            :total="total"
            :small="true"
            :pageSizeOptions="[10,20,30,40,50]"
            @update-currentPageNum="(e)=>currentPageNum=e"
            @update-pageSize="(e)=>pageSize=e"/>
  -->
  <!--分页-->
  <div class="my-pagination">

    <!--每页 共多少条数据-->
    <div class="left-views" v-if="!showMoreLittle">
      <span class="text-total">共</span>
      <span style="color: #676767;margin: 0 3px 0 5px">{{ total }}</span>
      <span class="text-total" style="margin-right: 10px"> 条</span>
      <!--输入设置页数-->
      <el-select v-model="pageSizeCopy" size="small" :disabled="disabledSelect"
                 :style="{width: getSelectAutoWidth+'px'}">
        <el-option v-for="item in pageSizeOptionsFilter" :key="item.value"
                   :label="item.label.length===1?('  '+item.label):(item.label)"
                   :value="item.value"/>
      </el-select>
      <span class="text-total-right" style="margin-left: 10px">条/页</span>
    </div>
    <!--pager-count官网最低只能传5 但此处我尝试传3，以便达到更好的缩小效果（有警告/但能用/可忽略）:current-page="currentPageNumCopy"-->

    <el-pagination
        :current-page="currentPageNumCopy"
        :page-size="pageSizeCopy"
        :total="total"
        :pager-count="small?3:5"
        :hide-on-single-page="true"
        :small="small"
        layout="prev, pager, next"
        @size-change="(e)=>pageSizeCopy=e"
        @current-change="currentChange">
    </el-pagination>

    <div class="right-views">
      <!--选择前往第几页  (e) => e>0?currentPageNumCopy=(e>total?total:e):currentPageNumCopy=1 -->
      <span class="text-page" style="margin-right: 10px" v-if="!showMoreLittle">前往</span>
      <el-input-number v-model="inputNumber" :min="1" :max="maxPageNum" size="small" controls-position="right"
                       @input="handleInput" v-if="!showMoreLittle">
      </el-input-number>
      <!--提示：由于v-model会更改当前页面，因此必须使用新变量，并添加一个事件来辅助修改（输入值为空的时候是null）
      而且不能使用change事件change需要按下回车键才触发(开发时候发现的，也许是一个bug)...-->
      <el-input-number v-model="inputNumber" :min="1" :max="maxPageNum" :controls="false" style="margin: 0 20px 0 10px;"
                       :style="{width: getInputAutoWidth+'px'}"
                       size="small" @input="handleInput" v-else>
      </el-input-number>
      <span class="text-page" style="margin-left: 10px" v-if="!showMoreLittle">页</span>
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
    small: {// 是否使用小尺寸的分页器
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
      return this.small || Math.ceil(this.total / this.pageSizeCopy) <= 1
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
    currentPageNum(newVal) {
      this.currentPageNumCopy = newVal
    },
    pageSize(newVal) {
      this.pageSizeCopy = newVal
    },
    currentPageNumCopy(newVal, oldVal) {
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
  align-items: center;
  flex-wrap: nowrap;
  background-color: #ffffff;
  width: 100%;
  height: 30px;
}

span {
  user-select: none;
  font-size: 13px;
//text-align: center;
}
</style>
