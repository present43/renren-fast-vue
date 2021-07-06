<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
            >Append</el-button
          >
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            >Delete</el-button
          >

          <el-button type="text" size="mini" @click="() => edit(data)"
            >Edit</el-button
          >
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      expandedKey: [],
      dialogType: "",
      dialogVisible: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
      },
    };
  },

  // 用来封装所有方法的
  methods: {
    // 获取菜单数据
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },

    append(data) {
      console.log(data);
      this.dialogVisible = true;
      this.dialogType = "add";
      this.title = "添加分类";
      this.category.parentCid = data.catId;
      // *1 是为了将字符串转换成为 int类型
      this.category.catLevel = data.catLevel * 1 + 1;

      // 重新設置输入框的值
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },

    edit(data) {
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.title = "修改分类";
      // 发送请求拿到最新数据 回写请求
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    // 添加三级分类
    addCategory() {
      console.log("提交的三级分类的数据：", this.category);

      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornParams(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单保存成功!",
        });
        // 关闭对话框
        (this.dialogVisible = false), this.getMenus();
        // 设置默认要展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },

    // 修改三级分类菜单
    editCategory() {
      // 解构拿到部分需要更新的参数
      var { name, catId, icon, productUnit } = this.category;
      // 组装赋值 key - v
      var data = {
        name: name,
        catId: catId,
        icon: icon,
        productUnit: productUnit,
      };

      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornParams(data, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });

        // 关闭对话框
        (this.dialogVisible = false), this.getMenus();
        // 设置默认要展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      var ids = [data.catId];

      this.$confirm(`是否删除【${data.name}】菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornParams(ids, false),
          }).then(({ data }) => {
            // 删除成功后刷新菜单数据

            this.$message({
              type: "success",
              message: "菜单删除成功!",
            });

            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          // this.$message({
          //   type: "info",
          //   message: "已取消删除",
          // });
        });
    },
  },

  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>