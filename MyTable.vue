<template>
  <div ref="table" style="display: flex;flex-direction: column;height: 100%;" :style="{width:tableWidth+'px',}">
    <!--1 本组件使用示例：
        <MyTable :tableData="tableData"
                 :columnList="columnList"/>
      -->
    <!--表格-->
    <el-table :data="showTableData">
      <el-table-column v-for="(item,index) in tableData[0]" :key="index" style="display:block;"
                       :prop="item" :label="firstLetterToUpperCase(item)" :width="columnWidth">
        <!--自定义表头-->
        <template #header>
          <div style="display: flex;align-items: center;">
            <!-- 可以在此添加你的图标 或者其他元素-->
            <el-dropdown :trigger="actionHover?'hover':'click'">
              <span @mouseenter="mouseEnter($event,index)" @mouseleave="mouseLeave($event,index)"
                    class="table-column-name"> {{ firstLetterToUpperCase(item) }}</span>
              <template #dropdown v-if="action[index]">
                <el-input @input="inputChange($event,index)" :style="{width: columnWidth-20+'px',}" size="small"
                          v-model="input[index]" clearable placeholder="Filter"/>
              </template>
            </el-dropdown>
          </div>
        </template>
      </el-table-column>
    </el-table>

    <MyPagination :currentPageNum="currentPageNum"
                  :pageSize="pageSize"
                  :total="total"
                  :pageSizeOptions="[10,20,30,40,50,100]"
                  @update-currentPageNum="(e)=>currentPageNum=e"
                  @update-pageSize="(e)=>pageSize=e"/>
  </div>

</template>

<script>
import MyPagination from "@/components/MyPagination.vue";
import MySelect from "@/components/MySelect.vue";


export default {
  name: "MyTable",// vue3主张组件名应当是多个单词组合，方便后期维护
  components: {MySelect, MyPagination},
  props: {
    tableData: {// 表格数据
      type: Array,
      default: () => []
    },
    columnList: {// 列名列表
      type: Array,
      default: () => ['1', '2']
    },
    width: {default: 1000},// 表格宽度
    columnWidth: {default: 120},// 每一列宽度
    columnNameStyle: {default: 0},// 列名的样式,0:默认,1:首字母大写,2:全部大写,3:全部小写
    multiple: {default: false},
  },
  data() {
    return {
      // table
      tableDataFilter: [],// 处理后的数据集
      pageSize: 10,// 每页显示的条/行数
      currentPageNum: 1,// 当前页码

      // Other
      input: [],
      action: [],
      actionHover: false,

    };
  },
  mounted() {
    const keys = this.tableData[0]
    console.log("keys:", keys, this.tableData[1])

    this.tableDataFilter = this.tableData[1]
  },
  computed: {
    total() {// 总条/行数
      // const w = this.tableWidth
      if (this.tableDataFilter)
        return this.tableDataFilter.length
      else
        return 0
    },
    tableWidth() {
      const w = (this.tableData[0].length) * (this.columnWidth)
      // console.log("宽度：", w)
      return w
    },

    showTableData() {
      if (this.tableDataFilter)
        return this.tableDataFilter.slice(this.pageSize * (this.currentPageNum - 1), this.pageSize * this.currentPageNum)
      else
        return []
    },

  },
  methods: {
    // 鼠标进入
    mouseEnter(e, i) {
      for (let j = 0; j < this.tableData[0].length; j++) {
        if (j === i) {
          this.action[i] = true
          continue
        }
        this.action[j] = false
      }
      this.actionHover = true
    },
    mouseLeave(e, i) {
      for (let j = 0; j < this.tableData[0].length; j++) {
        if (j === i) {
          this.action[i] = true
          continue
        }
        this.action[j] = false
      }
      this.actionHover = false
      // this.action[i] = false
    },
    inputChange(e, i) {
      // console.log("inputChange", e, i)
      // 进行搜索过滤：
      const query = e
      if (query === null || query === '') {// 没有输入值时，返回原有的下拉列表(all)
        this.tableDataFilter = this.tableData[1];
      } else {
        // console.log("filterMethod字符串存在非空:", query)
        // 通过对比 输入值query \ 下拉选项的label值 来进行筛选数组值
        this.tableDataFilter = this.tableData[1].filter((item) => {
          return item[this.tableData[0][i]].toLowerCase().includes(query.toLowerCase());// 包含匹配，而不是完全匹配
        });
      }
      // this.action = false
    },
    /*将列名的字符串首字母转换成大写
    * str: 要转换的字符串
    * return: 转换后的字符串*/
    firstLetterToUpperCase(str) {
      if (this.columnNameStyle === 1) {
        return this.firstLetterToUpperCase(str);
      } else if (this.columnNameStyle === 2) {
        return str.toUpperCase();
      } else if (this.columnNameStyle === 3) {
        return str.toLowerCase();
      } else {
        return str;
      }
    },

  },
  watch: {
    valueListCopy(newVal) {
      // console.log("父组件传来新值/valueListCopy = ", newVal);
      this.valueListCopy = newVal;
      this.total = this.valueListCopy?.length;
      this.currentPageNum = 1;
    },

  }
};
</script>

<style scoped>

.elPaginationSelect .el-input input {
  border: none !important;
  width: 100%;
}

/* 鼠标悬浮在列名上方触发的背景颜色*/
.table-column-name:hover {
  background-color: pink;
}

</style>
