<template>
  <el-tree
    :data="data"
    :props="defaultProps"
    @node-click="handleNodeClick"
  ></el-tree>
</template>

<script>
export default {
  data () {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
        this.$http({
            url: this.$http.adornUrl('/product/category/list/tree'),
            method: 'get',
        }).then((res) => {
            this.data = res.data.page
        })
    }
  },
  mounted () {
    this.getMenus()
  }
}
</script>
