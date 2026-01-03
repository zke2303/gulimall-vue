<template>
  <el-tree
    :data="data"
    :props="defaultProps"
    :expand-on-click-node="false"
    :show-checkbox="true"
    node-key="catId"
    :default-expanded-keys="expandedKey"
  >
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button
          type="text"
          size="mini"
          @click="() => append(data)"
          v-if="node.level <= 2"
        >
          Append
        </el-button>
        <el-button
          type="text"
          size="mini"
          @click="() => remove(node, data)"
          v-if="node.childNodes.length == 0"
        >
          Delete
        </el-button>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(res => {
        this.data = res.data.page;
      });
    },
    append(data) {},

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除当${data.name}前菜单, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(res => {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            // 删除节点后，刷新出新数据
            this.getMenus();
            // 展开删除节点的父节点
            this.expandedKey = [node.parent.data.catId]
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  },
  mounted() {
    this.getMenus();
  }
};
</script>
