# vue3-my-ElementPlus

## 置顶|Top
| 使用案例：

1.0 定义表格数据（测试使用）
```js
data() {
  return {
    tableData: [],
    value:[],
    valueList: [],
  };
},

// 构造表格测试数据
    // 1 第一行：列名，该行是一个数组
    let columns = []
    for (let i = 0; i < 10; i++) {
      columns[i] = 'column' + i
    }
    // 2 其他行：表格数据值，每一行的一个对象
    let list = [];
    for (let j = 0; j < 9999; j++) {
      // 每一行的数据对象
      let obj = {}
      for (let i = 0; i < columns.length; i++) {
        obj[columns[i]] = i + 'value' + j
      }
      list.push(obj);// 存储每一行到数组
    }
    this.tableData = [columns, list] // 表格数据请传入这个数组
    console.log('构造表格数据：', list, "\ncolumns:", columns)

    // 获取某一列的数据，提供给MySelect组件：
    let i = 6;// 假设获取第6列的数据
    this.valueList = list.map(obj => ({
      value: obj[columns[i-1]],
      label: obj[columns[i-1]]
    }));
```
1.1 MyTable

```html
<MyTable :table-data="tableData"/>
```

1.2 MySelect
```html
// 其中 initValue 为初始化数据，必须传入一个数组，形式为["","","",...]
// 其中每一个字符串的值是 valueList 数组的 value（当然，也可以不是）
<MySelect :initValue="[]"
          :valueList="valueList"
          :placeholder="'placeholder'"
          :multiple="true"
          @selectorChanged="(e)=>value=e"
          @selectorCleared="(e)=>value=e"/>
```

1.3 MyPagination
```html
一般不独立使用，已经内置在上面两个组件中。
```
---

## 细节说明：
| 门槛：
- 本仓库的组件依赖于 `Element UI` 或者 `Element Plus`的组件，需要用户已经熟悉使用相关组件
- 理论上支持`vue2`、`vue3`（开发过程中使用选项式API风格，并尽量避免vue3新语法）
- 此外，表格`MyTable`、下拉选择`MySelect` 都依赖的自定义的分页组件 `MyPagination`

---

| 使用场景：
- 表格数据分析很常用，我们常使用```Table```
- 对于表格中的每一列，我们常使用`Select`
- 对于数量比较大情况，我们常使用`Pagination`

---

| 效果展示：

1.1 MyTable

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f095cf587d746fabc8fc0f5318a2d72.gif)


1.2 MySelect

![在这里插入图片描述](https://img-blog.csdnimg.cn/75c8389c5949480095d2b4ff5d2a2ab0.gif)


1.3 MyPagination

入上图所示，分为两种尺寸。
如上表格MyTable是大尺寸，可以显示更多信息
下拉选择框MySelect小尺寸，仅可以跳转。

----

| 实现功能：

1.1 MyTable
- ```底部分页```的展示、跳转功能
- 自定义```标题```（支持比官网更灵活的自定义）
- 自定义```过滤```功能

1.2 MySelect
-  ```底部分页```跳转功能
 - 自定义```过滤```功能
 - 自定义`单选 多选`逻辑

1.3 MyPagination
-  `大 小`两种尺寸，大的适合表格，小的适合选择框
 - `数据总数`展示、`每一页多少行`数据、页面`跳转`（可输入）
