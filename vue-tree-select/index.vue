<template>
  <select-tree
    :width="`100%`"
    :height="`auto`"
    :maxHeight="`400px`"
    :nodeKey="'department_id'"
    size=""
    multiple
    clearable
    :defaultProps="{
      children: 'children',
      label: 'short_name'
    }"
    :defaultExpandedKeys="`默认展开的数据`"
    :checkedKeys="`选中的数组`"
    @change="changeTreeItem"
    :getGroupSequence="`获取treeData的数据请求`"
  />
</template>
<script>
import selectTree from "./selectTree";
export default {
  data() {

    return {
    };
  },
  components: {
    selectTree
  },
  props: {
  },
  watch: {

  },
  created() {},
  mounted() {},
  methods: {
    changeTreeItem(res){
      console.log(res);
    }
  }
};
</script>
<style lang="scss" scoped>
.flex1 {
  flex: 1;
}
</style>
