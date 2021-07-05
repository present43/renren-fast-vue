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
      </span>
    </span>
  </el-tree>

<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  :before-close="handleClose">

  <el-form :model="category">
    <el-form-item label="分类名称">
      <el-input v-model="category.name" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>

  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCategory">确 定</el-button>
  </span>
</el-dialog>

</div>

</template>

<script>
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandedKey: [],
      dialogVisible:false,
      category:{name:"" ,parentCid: 0,catLevel: 0,showStatus: 1, sort: 0}
    }
  },

  // 用来封装所有方法的
  methods: {
    // 获取菜单数据
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        this.menus = data.data
      })
    },

    append (data) {
      console.log(data)
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      // *1 是为了将字符串转换成为 int类型
      this.category.catLevel = data.catLevel*1 + 1;

    },

    // 添加三级分类
    addCategory(){
      console.log("提交的三级分类的数据：",this.category);

      this.$http({
      url: this.$http.adornUrl('/product/category/save'),
      method: 'post',
      data: this.$http.adornParams(this.category,false)
      }).then( ({data}) => { 
            this.$message({
              type: 'success',
              message: '菜单保存成功!'
            })
            // 关闭对话框
            this.dialogVisible = false,
            this.getMenus();
            // 设置默认要展开的菜单
            this.expandedKey = [this.category.parentCid];
      })
     
    },

    remove (node, data) {
      var ids = [data.catId]

      this.$confirm(`是否删除【${data.name}】菜单？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornParams(ids, false)
          }).then(({ data }) => {
            // 删除成功后刷新菜单数据

            this.$message({
              type: 'success',
              message: '菜单删除成功!'
            })

            this.getMenus()
            this.expandedKey = [node.parent.data.catId]
          })
        })
        .catch(() => {
          // this.$message({
          //   type: "info",
          //   message: "已取消删除",
          // });
        })
    }
  },

  created () {
    this.getMenus()
  }
}
</script>

<style>
</style>