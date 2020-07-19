<!--
 * @description: 3级分类组件
 * @AUthor: 孙启新
 * @Date: 2020-07-18 09:26:56
 * @LastEditTime: 2020-07-18 14:28:55
 * @FilePath: \gulimall-view\renren-fast-vue\src\views\modules\common\category.vue
-->
<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    highlight-current
    ref="tree"
    @node-click="nodeClick"
  >
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    /**
     * @description: 获取分类数据的请求
     * @param {type}
     * @return:
     */
    getMenus() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    /**
     * @description: 当节点被点击触发事件，向父组件发送事件
     * @param {type}
     * @return:
     */
    nodeClick(data, node, component) {
      if (node.level == 3) {
        this.$emit("tree-node-click", data, node, component);
      }
    }
  },
  created() {
    //发送请求获取分类数据
    this.getMenus();
  }
};
</script>

<style></style>
