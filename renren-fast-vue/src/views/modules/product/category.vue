<template>
  <div>
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
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="活动名称" >
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      expandedKey: [],
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
      },  
      dialogVisible: false,
      defaultProps: {
        children: "children",
        label: "name"
      }
    }
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
        this.data = res.data.page
      })
    },

    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done()
        })
        .catch(_ => {})
    },

    append(data) {
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 - 1
    },

    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(res => {
        this.$message({
          type: "success",
          message: "添加成功"
        })
        // 添加成功后，取消dilog框
        this.dialogVisible = false
        // 重新加载数据
        this.getMenus()
        // 展开父节点
        this.expandedKey = [this.category.parentCid]
        // 清空category数据
        this.category = ""
      })
    },

    remove(node, data) {
      var ids = [data.catId]
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
            })
            // 删除节点后，刷新出新数据
            this.getMenus()
            // 展开删除节点的父节点
            this.expandedKey = [node.parent.data.catId]
          })
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          })
        })
    }
  },
  mounted() {
    this.getMenus()
  }
}
</script>
