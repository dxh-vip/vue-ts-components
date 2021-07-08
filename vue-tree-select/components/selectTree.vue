<template>
  <div>
    <div
      v-if="isShowSelect"
      class="mask"
      @click="isShowSelect = !isShowSelect"
    />
    <el-popover
      v-model="isShowSelect"
      placement="bottom-start"
      :width="popoverWidth"
      trigger="manual"
      @hide="popoverHide"
    >
      <el-tree
        ref="tree"
        class="common-tree"
        :data="treeData"
        lazy
        :load="loadNode"
        :style="style"
        :props="defaultProps"
        :show-checkbox="multiple"
        :node-key="nodeKey"
        :check-strictly="checkStrictly"
        :default-expanded-keys="defaultExpand"
        :default-checked-keys="defaultDeptAll"
        :highlight-current="true"
        :render-after-expand="true"
        @node-click="handleNodeClick"
        @check-change="handleCheckChange"
      />
      <el-select
        slot="reference"
        ref="select"
        v-model="defaultDeptAll"
        :style="selectStyle"
        :size="size"
        :multiple="multiple"
        :clearable="clearable"
        :collapse-tags="collapseTags"
        class="tree-select"
        @click.native="isShowSelect = !isShowSelect"
        @remove-tag="removeSelectedNodes"
        @clear="removeSelectedNode"
        @change="changeSelectedNodes"
      >
        <el-option
          v-for="item in options"
          :key="item.department_id"
          :label="item.short_name"
          :value="item.department_id"
        />
      </el-select>
    </el-popover>
  </div>
</template>

<script>
export default {
  props: {
    getGroupSequence: {
      type: Function,
      default() {}
    },
    defaultProps: {
      type: Object,
      default() {
        return {};
      }
    },
    // 配置是否可多选
    multiple: {
      type: Boolean,
      default() {
        return false;
      }
    },
    clear: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 配置是否可清空选择
    clearable: {
      type: Boolean,
      default() {
        return false;
      }
    },

    // 配置多选时是否将选中值按文字的形式展示
    collapseTags: {
      type: Boolean,
      default() {
        return false;
      }
    },
    nodeKey: {
      type: String,
      default() {
        return "department_id";
      }
    },
    // 显示复选框情况下，是否严格遵循父子不互相关联
    checkStrictly: {
      type: Boolean,
      default() {
        return false;
      }
    },
    // 默认选中的节点key数组
    checkedKeys: {
      type: Array,
      default() {
        return [];
      }
    },
    size: {
      type: String,
      default() {
        return "medium";
      }
    },
    width: {
      type: String,
      default() {
        return `250px`;
      }
    },
    height: {
      type: String,
      default() {
        return `300px`;
      }
    },
    defaultExpandedKeys: {
      type: Array,
      default() {
        return ["D1000001"];
      }
    },
    maxHeight: {
      type: String,
      default() {
        return `400px`;
      }
    }
  },
  data() {
    return {
      treeData: [],
      isShowSelect: false, // 是否显示树状选择器
      options: [], // select选中项创建
      selectedData: [], // 选中的节点
      defaultDeptAll: [], // 默认选中的节点数组
      defaultExpand: [],
      popoverWidth: 0,
      firstLoad: false,
      style:
        "width:" +
        this.width +
        ";" +
        "height:" +
        this.height +
        ";max-height:" +
        this.maxHeight +
        ";",
      selectStyle: "width:" + this.width + ";",
      checkedIds: [],
      checkedData: []
    };
  },
  watch: {
    isShowSelect() {
      // 隐藏select自带的下拉框
      this.$refs.select.blur();
    },
    defaultExpandedKeys(val) {
      console.log("defaultExpandedKeys", val);
      if (!val && this.clear) return;
      this.defaultExpand = val;
    },
    checkedKeys(val) {
      console.log("init", val, this.clear);
      if (!val && this.clear) return;
    }
  },
  mounted() {
    this.getTreeData(() => {
      this.initCheckedData();
    });
  },
  methods: {
    // 单选时点击tree节点，设置select选项
    setSelectOption(node) {
      const tmpMap = {};
      tmpMap.value = node.key;
      tmpMap.label = node.label;
      this.options = [];
      this.options.push(tmpMap);
      this.selectedData = node.key;
    },
    // 单选，清空选中
    clearSelectedNode() {
      this.selectedData = [];
      this.$refs.tree.setCurrentKey(null);
    },
    // 多选，清空所有勾选
    clearSelectedNodes() {
      var checkedKeys = this.$refs.tree.getCheckedKeys(); // 所有被选中的节点的 key 所组成的数组数据
      for (let i = 0; i < checkedKeys.length; i++) {
        this.$refs.tree.setChecked(checkedKeys[i], false);
      }
    },
    getTreeData(cb) {
      this.getGroupSequence({ department_id: "D1000001" }).then(res => {
        this.treeData = res.data;
        cb();
      });
    },
    initCheckedData() {
      if (this.multiple) {
        // 多选
        this.defaultExpand = this.defaultExpandedKeys;
        this.selectedData = this.checkedKeys;
        this.options = this.checkedKeys;
        this.checkedKeys.map(item => {
          if (item) {
            this.defaultDeptAll.push(item.department_id);
          }
        });
      } else {
        // 单选
        if (this.selectedData.length > 0) {
          this.checkSelectedNode(this.selectedData);
        }
      }
      this.$nextTick(() => {
        this.popoverWidth = this.$refs.select.$el.clientWidth - 24;
      });
    },
    popoverHide() {
      this.$emit("popoverHide", this.checkedIds, this.checkedData);
    },
    // 单选，节点被点击时的回调,返回被点击的节点数据
    handleNodeClick(data, node) {
      if (!this.multiple) {
        this.setSelectOption(node);
        this.isShowSelect = !this.isShowSelect;
        this.$emit("change", this.selectedData);
      }
    },
    // 多选，节点勾选状态发生变化时的回调
    handleCheckChange(data, checked, node) {
      console.log("handleCheckChange", data, checked, node);
      if (checked) {
        let array = this.$refs.tree.getNode(data.department_id);
        if (array.childNodes.length == 0) {
          this.checkedData.push(data);
          this.defaultDeptAll.push(data.department_id);
        }
      } else {
        if (this.checkedData.length > 0) {
          this.checkedData.forEach((item, index) => {
            if (item.department_id == data.department_id) {
              this.defaultDeptAll.splice(index, 1);
              this.checkedData.splice(index, 1);
            }
          });
        }
      }
      if (this.firstLoad || this.checkedKeys.length == 0) {
        this.setCheckedKey();
      } else {
        this.firstLoad = true;
      }
      this.$emit("change", this.defaultDeptAll, this.defaultExpand);
    },
    // 多选,删除任一select选项的回调
    removeSelectedNodes(val) {
      this.$refs.tree.setChecked(val, false);
      var node = this.$refs.tree.getNode(val);
      if (!this.checkStrictly && node.childNodes.length > 0) {
        this.treeToList(node).map(item => {
          if (item.childNodes.length <= 0) {
            this.$refs.tree.setChecked(item, false);
          }
        });
      }
      this.$emit("change", this.selectedData);
    },
    treeToList(tree) {
      var queen = [];
      var out = [];
      queen = queen.concat(tree);
      while (queen.length) {
        var first = queen.shift();
        if (first.childNodes) {
          queen = queen.concat(first.childNodes);
        }
        out.push(first);
      }
      return out;
    },
    // // 单选,清空select输入框的回调
    removeSelectedNode() {
      this.clearSelectedNode();
      this.$emit("change", this.selectedData);
    },
    // 选中的select选项改变的回调
    changeSelectedNodes(selectedData) {
      // // 多选,清空select输入框时，清除树勾选
      if (this.multiple && selectedData.length <= 0) {
        this.clearSelectedNodes();
      }
      this.$emit("change", this.selectedData);
    },
    setCheckedKey() {
      let treeData = this.$refs.tree.getCheckedNodes();
      console.log(treeData);
      // 过滤全选的bug
      let temp = JSON.parse(JSON.stringify(treeData));
      let result = temp.filter(arr => {
        const parentId = arr.parent_id;
        const curIdArr = treeData.filter(ele => {
          return ele.department_id == parentId;
        });
        return curIdArr.length == 0;
      });
      let array = this.$refs.tree.getHalfCheckedKeys();
      result.forEach(item => {
        array.concat(item.parent_id);
      });
      console.log("end", array, result);
      this.options = this.selectedData = result;
      console.log("options", this.options, result);
      this.defaultDeptAll = result.map(item => item.department_id);
      this.defaultExpand = array;
    },
    loadNode(node, resolve) {
      if (node.level === 0) {
        return;
      }
      this.getGroupSequence({ department_id: node.data.department_id }).then(
        res => {
          if (res.data && res.data.length > 0) {
            resolve(res.data);
            this.$refs.tree.setCheckedKeys(this.defaultDeptAll);
          } else {
            resolve([]);
          }
        }
      );
    }
  }
};
</script>

<style scoped>
.mask {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  opacity: 0;
  z-index: 11;
}

.common-tree {
  overflow: auto;
}

.tree-select {
  z-index: 111;
}
</style>
