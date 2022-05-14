# DataTable for Vue3
- Bootstrap5 is required

# Features
- You can toggle sort by clicking on the column head
- You can handle the events (**onPageNext / onPagePrev**) - insure to set **handleServer** to **true** when you get the data from server
- Pagination
- Custom cell render
- You can select columns you want to render
- You can set a custom color for focused row
- You can search on multiple columns
- Columns can be removed, added to filter to search
- You can make custom search hndlaer "**onSearch**" this prop is a function (query, cols) => {}

# Usage 

```vue
<template>
  <div id="cont">
    <DataTable
      :data="[
        { a: 10, b: 55 },
        { a: 9, b: 32 },
        { a: 66, b: 213 },
      ]"
      :cols="[
        { name: '_dataTableSortNumber', title: 'No.', width: '100px' },
        { name: 'a', title: 'First Col' },
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
