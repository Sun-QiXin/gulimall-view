<!--
 * @description: 分类管理
 * @AUthor: 孙启新
 * @Date: 2020-07-14 12:32:56
 * @LastEditTime: 2020-07-15 16:21:18
 * @FilePath: \renren-fast-vue\src\views\modules\product\category.vue
-->
<template>
  <div>
    <el-row style="margin-bottom: 15px;">
      <el-switch
        v-model="isDrag"
        active-color="#13ce66"
        inactive-color="#ff4949"
        active-text="激活拖拽"
        inactive-text="关闭拖拽"
        style="padding-right: 15px;"
      >
      </el-switch>
      <el-button
        v-if="isDrag"
        type="success"
        size="small "
        plain
        @click="saveDrag()"
        :disabled="updateNodes.length <= 0"
        >保存当前布局
      </el-button>
      <el-button
        v-if="isDrag"
        type="success"
        size="small "
        plain
        @click="notSaveDrag()"
        :disabled="updateNodes.length <= 0"
        >取消保存
      </el-button>
      <el-button type="danger" size="small " plain @click="deleteAllSelect()"
        >批量删除
      </el-button>
    </el-row>

    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      highlight-current
      :default-expanded-keys="expandedKey"
      :icon-class="categoryForm.icon"
      :draggable="isDrag"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="tree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level != 3"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => update(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="dialogTitle"
      :visible.sync="centerDialogVisible"
      width="30%"
      center
    >
      <el-form
        label-position="right"
        ref="categoryForm"
        :rules="rules"
        label-width="45px"
        :model="categoryForm"
        class="demo-ruleForm"
        status-icon
      >
        <el-form-item label="名称:" prop="name">
          <el-input
            placeholder="请输入分类名称！"
            v-model="categoryForm.name"
            prefix-icon="el-icon-edit-outline"
            clearable
            autocomplete="off"
          >
          </el-input>
        </el-form-item>
        <el-form-item label="图标:">
          <el-input
            placeholder="请输入图标名称！"
            v-model="categoryForm.icon"
            prefix-icon="el-icon-edit-outline"
            clearable
            autocomplete="off"
          >
          </el-input>
        </el-form-item>
        <el-form-item label="单位:">
          <el-input
            placeholder="请输入计量单位！"
            v-model="categoryForm.productUnit"
            prefix-icon="el-icon-edit-outline"
            clearable
            autocomplete="off"
          >
          </el-input>
        </el-form-item>
        <el-form-item label="排序:">
          <el-input
            placeholder="数组越大等级优先级越低，默认0"
            v-model="categoryForm.sort"
            prefix-icon="el-icon-edit-outline"
            clearable
            autocomplete="off"
          >
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="centerDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="categorySubmit()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    /**
     * @description: 表单验证
     * @param {type}
     * @return:
     */
    var validateName = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("名称不能为空！"));
      } else {
        callback();
      }
    };

    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      },
      expandedKey: [],
      centerDialogVisible: false,
      dialogType: "",
      dialogTitle: "",
      maxLevel: 0,
      updateNodes: [],
      categoryForm: {
        catId: "",
        name: "",
        parentCid: 0,
        catLevel: 1,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: ""
      },
      rules: {
        name: [{ validator: validateName, trigger: "blur" }]
      },
      isDrag: false,
      nowParentCidS: []
    };
  },
  methods: {
    /**
     * @description: 点击添加分类按钮
     * @param {type}
     * @return:
     */
    append(data) {
      this.dialogTitle = "添加分类";
      this.dialogType = "insertCategory";
      this.centerDialogVisible = true;
      this.categoryForm.parentCid = data.catId;
      this.categoryForm.catLevel = data.catLevel * 1 + 1;
      this.categoryForm.catId = "";
      this.categoryForm.name = "";
      this.categoryForm.showStatus;
      this.categoryForm.sort = 0;
      this.categoryForm.icon = "";
      this.categoryForm.productUnit = "";
    },
    /**
     * @description: 点击删除分类按钮
     * @param {type}
     * @return:
     */
    remove(node, data) {
      this.$confirm(`此操作将永久删除分类【${data.name}】, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              showClose: true,
              message: "删除成功！",
              type: "success"
            });
            //重新请求数据
            this.getMenus();
            //设置展开刚刚删除的那一级菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            showClose: true,
            type: "info",
            message: "已取消删除!"
          });
        });
    },
    /**
     * @description: 添加分类到数据库方法
     * @param {type}
     * @return:
     */
    insertCategory() {
      console.log(this.categoryForm);

      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.categoryForm, false)
      }).then(({ data }) => {
        this.$message({
          showClose: true,
          message: "添加成功！",
          type: "success"
        });
        //重新请求数据
        this.getMenus();
        this.centerDialogVisible = false;
        //设置展开刚刚删除的那一级菜单
        this.expandedKey = [this.categoryForm.parentCid];
      });
    },
    /**
     * @description: 点击修改分类按钮
     * @param {type}
     * @return:
     */
    update(data) {
      this.dialogTitle = "修改分类";
      this.dialogType = "updateCategory";
      this.centerDialogVisible = true;
      //发送请求获取到菜单的最新数据再展示
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        this.categoryForm.catId = data.data.catId;
        this.categoryForm.parentCid = data.data.parentCid;
        this.categoryForm.name = data.data.name;
        this.categoryForm.productUnit = data.data.productUnit;
        this.categoryForm.icon = data.data.icon;
      });
    },
    /**
     * @description: 修改分类数据到数据库方法
     * @param {type}
     * @return:
     */
    updateCategory() {
      //解构出要修改的项
      let { catId, name, icon, sort, productUnit } = this.categoryForm;
      let data = { catId, name, icon, sort, productUnit };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.$message({
          showClose: true,
          message: "修改成功！",
          type: "success"
        });
        //重新请求数据
        this.getMenus();
        this.centerDialogVisible = false;
        //设置展开刚刚删除的那一级菜单
        this.expandedKey = [this.categoryForm.parentCid];
      });
    },
    /**
     * @description: 点击确定提交按钮
     * @param {type}
     * @return:
     */
    categorySubmit() {
      this.$refs.categoryForm.validate(valid => {
        //提交表单时再次验证
        if (valid) {
          if (this.dialogType == "updateCategory") {
            this.updateCategory();
          } else if (this.dialogType == "insertCategory") {
            this.insertCategory();
          }
        } else {
          this.$message({
            showClose: true,
            message: "名称不能为空，请重新输入！",
            type: "error"
          });
          return false;
        }
      });
    },
    /**
     * @description: 判断是否可以拖拽进该节点
     * @param {type}
     * @return:
     */
    allowDrop(draggingNode, dropNode, type) {
      //要求：被拖动的当前节点以及父节点总层数不能大于3层
      this.countNodeLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      //console.log(`当前拖动节点有 ${deep} 层`);

      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    /**
     * @description: 求出当前所拖动的节点一共多少层
     * @param {type}
     * @return:
     */
    countNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let index = 0; index < node.childNodes.length; index++) {
          if (node.childNodes[index].level > this.maxLevel) {
            this.maxLevel = node.childNodes[index].level;
          }
          this.countNodeLevel(node.childNodes[index]);
        }
      }
    },
    /**
     * @description: 拖拽成功后将数据保存进数组
     * @param {type}
     * @return:
     */
    handleDrop(draggingNode, dropNode, dropType, ev) {
      //console.log("tree drop: ", draggingNode, dropNode, dropType);
      //1、获取当前节点最新的父节点id
      let nowParentCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        //父节点id
        nowParentCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        //获取当前拖拽进的节点的数组信息，用于排序
        siblings = dropNode.parent.childNodes;
      } else {
        //父节点id
        nowParentCid = dropNode.data.catId;
        //获取当前拖拽进的节点的数组信息，用于排序
        siblings = dropNode.childNodes;
      }

      //将所有父节点的id存进数组
      this.nowParentCidS.push(nowParentCid);

      //2、获取当前节点最新的排序以及当前节点最新的层级
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前拖拽的节点就更新下他的父id

          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            //说明层级发生了变化
            catLevel = siblings[i].level;
            //修改他子节点的层级
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: nowParentCid
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      console.log(this.updateNodes);
    },
    /**
     * @description: 发送请求将拖拽更改的数据发送到数据库
     * @param {type}
     * @return:
     */
    saveDrag() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          showClose: true,
          message: "修改成功！",
          type: "success"
        });
        //将数据清空
        this.updateNodes = [];
        this.maxLevel = 0;
        //重新请求数据
        this.getMenus();
        //设置展开修改过的父菜单
        this.expandedKey = this.nowParentCidS;
        //设置展开修改过的父菜单数据清空
        this.nowParentCidS = [];
      });
    },
    /**
     * @description: 取消保存拖拽的数据，清空数组
     * @param {type}
     * @return:
     */
    notSaveDrag() {
      //将数据清空
      this.updateNodes = [];
      this.maxLevel = 0;
      //重新请求数据
      this.getMenus();
      this.$message({
        showClose: true,
        message: "取消保存成功！",
        type: "success"
      });
    },
    /**
     * @description: 递归修改子节点层级
     * @param {type}
     * @return:
     */
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let currentNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: currentNode.catId,
            catLevel: node.childNodes[i].level
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    /**
     * @description: 批量删除选中的分类
     * @param {type}
     * @return:
     */
    deleteAllSelect() {
      let selectNodeS = this.$refs.tree.getCheckedNodes();
      let selectIdS = [];
      let selectParentIdS = [];
      let selectParentNames = [];
      for (const node of selectNodeS) {
        //取得所有父菜单id
        selectParentIdS.push(node.parentCid);
        //遍历取出所有的id
        selectIdS.push(node.catId);
        //遍历取出所有的name
        selectParentNames.push(node.name);
      }
      //取出父菜单最小的那个值
      let minParentId = Math.min.apply(null, selectParentIdS);
      console.log(selectIdS);

      this.$confirm(
        `此操作将永久删除【${selectParentNames.join("、")}】分类, 是否继续?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(selectIdS, false)
          }).then(({ data }) => {
            this.$message({
              showClose: true,
              message: "删除成功！",
              type: "success"
            });
            //重新请求数据
            this.getMenus();
            //设置展开刚刚删除的那一级菜单
            this.expandedKey = [minParentId];
          });
        })
        .catch(() => {
          this.$message({
            showClose: true,
            type: "info",
            message: "已取消删除!"
          });
        });
    },
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
    }
  },
  created() {
    //发送请求获取分类数据
    this.getMenus();
  }
};
</script>

<style>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
