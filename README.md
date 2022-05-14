# DataTable for Vue3
  DataTable is an easy way to handle your rows in a table with the ability to customize your columns and rows

# Requirements
- Bootstrap5 is required
- Bootstrap icons

# Features
- You can toggle sort by clicking on the column head
- You can handle the events (**onPageNext / onPagePrev**) - insure to set **handleServer** to **true** when you get the data from server
- Pagination
- Custom cell render
- You can select columns you want to render
- You can set a custom color for focused row
- You can search on multiple columns
- Columns can be removed, added to filter to search
- Columns can be toggle all (add all columns, remove all columns from filter)
- You can customize if you want to display the icon of toggle all columns or not using **showToggleAllFilters** prop
- You can make custom search handler "**onSearch**" prop, this prop is a function **(query, cols) => {}**
- Neasted Objects (search / value getter)

# Usage 
- First you need to add bootstrap5 css to your html header page
  ```html
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" crossorigin="anonymous">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.2/font/bootstrap-icons.css">
  ```
- Code example

```vue
<template>
  <div id="cont">
    <DataTable
      :data="[
        { a: 10, b: 55, child: { age: 10 } } },
        { a: 9, b: 32, child: { age: 12 } },
        { a: 66, b: 213, child: { age: 33 } },
      ]"
      :cols="[
        { name: '_dataTableSortNumber', title: 'No.', width: '100px' },
        { name: 'a', title: 'First Col' },
        { name: 'child.age', title: 'Child Age' },
        {
          name: 'b',
          title: 'Second Col',
          render: (row) => renderSeondColumn(row),
        },
      ]"
      :rowsPerPage="1"
      :onPageNext="null"
      :onPagePrev="null"
      :rowsCount="null"
      :serverHandle="null"
      :focusedRowColor="null"
      :onSearch="null"
      :onSearch="(query, cols) => serverSearch(query, cols)"
      :onSearch="serverSearch"
      :showToggleAllFilters="true"
    />
  </div>
</template>

<script>
import DataTable from "@/components/DataTable.vue";
export default {
  components: { DataTable },
  methods: {
    serverSearch(query, cols) {
      console.log(query, cols);
    },
    renderSeondColumn(row) {
      return (
        <div style="color: red" onClick={() => alert(row.b)}>
          {row.b}
        </div>
      );
    },
  },
};
</script>

<style>
#cont {
  width: 100%;
}
</style>
```
