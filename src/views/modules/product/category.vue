<template>
  <div class="category">
    <el-row>
      <el-button type="primary" v-on:click="addLevel1Catagory"
        >添加一级分类</el-button
      >
      <el-button type="danger" v-on:click="branchDelete">批量删除</el-button>
    </el-row>

    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKeys"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%"
    >
      <el-form :model="category" :rules="rules" ref="category">
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标地址" prop="icon">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" prop="productUnit">
          <el-select
            v-model="category.productUnit"
            placeholder="请选择计量单位"
          >
            <el-option
              v-for="item in productUnits.list"
              :key="item.dicCode"
              :label="item.dicName"
              :value="item.dicName"
            >
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData('category')"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 1,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        productCount: 0,
      },
      dialogVisible: false,
      menus: [],
      productUnits: [],
      expandedKeys: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      submitType: "",
      title: "",
      rules: {
        name: [{ required: true, message: "请输入分类名称", trigger: "blur" }],
        icon: [{ required: true, message: "请输入图标地址", trigger: "blur" }],
        productUnit: [
          { required: true, message: "请选择计量单位", trigger: "change" },
        ],
      },
    };
  },
  // 方法集合
  methods: {
    // 获取菜单方法
    getMeuns() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("获取菜单：", data.page);
        this.menus = data.page;
      });
    },
    // 获取计量单位
    getProductUnits() {
      this.$http({
        url: this.$http.adornUrl("/product/dicinfo/list"),
        method: "get",
      }).then(({ data }) => {
        console.log("获取计量单位：", data.page);
        this.productUnits = data.page;
        console.log("productUnits：", this.productUnits);
      });
    },

    submitData(formName) {
      //表单验证
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (this.submitType == "append") {
            this.addCategory();
          }
          if (this.submitType == "edit") {
            this.editCategory();
          }
          if (this.submitType == "addLevel1") {
            this.addLevel1Category();
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },

    append(data) {
      console.log("append", data);
      //调用获取计量单位方法
      this.getProductUnits();
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.submitType = "append";
      this.title = "添加分类";
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
    },

    //添加三级分类
    addCategory() {
      console.log("数据：", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功!",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新出新的菜单
        this.getMeuns();
        //设置需要默认展开的菜单
        this.expandedKeys = [this.category.parentCid];
      });
    },

    remove(node, data) {
      console.log("remove", node, data);

      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            console.log("删除返回数据：", data);
            this.$message({
              message: "菜单删除成功!",
              type: "success",
            });
            //刷新出新的菜单
            this.getMeuns();
            //设置需要默认展开的菜单
            this.expandedKeys = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    edit(data) {
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        // data: this.$http.adornData(data.catId, false),
      }).then(({ data }) => {
        console.log("修改：", data.category);
        //回显数据
        this.category.catId = data.category.catId;
        this.category.name = data.category.name;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid; //用于修改完成后展开默认菜单
        this.submitType = "edit";
        this.title = "修改分类";
        //调用获取计量单位方法
        this.getProductUnits();
      });
    },

    //修改分类
    editCategory() {
      console.log("数据：", this.category);
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功!",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新出新的菜单
        this.getMeuns();
        //设置需要默认展开的菜单
        this.expandedKeys = [this.category.parentCid];
      });
    },

    //批量删除
    branchDelete() {
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }
      this.$confirm("是否批量删除选中菜单", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            console.log("删除返回数据：", data);
            this.$message({
              message: "菜单删除成功!",
              type: "success",
            });
            //刷新出新的菜单
            this.getMeuns();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消批量删除",
          });
        });
    },

    //展示添加一级分类对话框
    addLevel1Catagory() {
      this.dialogVisible = true;
      this.category.catLevel = 1;
      this.category.parentCid = 0;
      this.submitType = "addLevel1";
      //调用获取计量单位方法
      this.getProductUnits();
      this.title = "添加一级分类";
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
    },

    //添加一级分类
    addLevel1Category() {
      console.log("数据：", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功!",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新出新的菜单
        this.getMeuns();
      });
    },
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMeuns();
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, // 生命周期 - 创建之前
  beforeMount() {}, // 生命周期 - 挂载之前
  beforeUpdate() {}, // 生命周期 - 更新之前
  updated() {}, // 生命周期 - 更新之后
  beforeDestroy() {}, // 生命周期 - 销毁之前
  destroyed() {}, // 生命周期 - 销毁完成
  activated() {}, // 如果页面有keep-alive缓存功能，这个函数会触发
};
</script>

<style lang="scss" scoped>
//有scoped表示样式只在当前组件生效，引用的别人的组件都修改不了，必须使用  /deep/ 才能修改

// /deep/ .category .el-form-item.el-form-item--medium .el-form-item__label {
//   text-align: left;
//   float: none;
// }

// /deep/ .category .el-form-item.el-form-item--medium .el-select.el-select--medium {
//   display: block;
// }
</style>

<style>
/* 不加scope 代表样式全局生效  所有组件生效 */
.category .el-form-item.el-form-item--medium .el-form-item__label {
  text-align: left;
  float: none;
}

.category .el-form-item.el-form-item--medium .el-select.el-select--medium {
  display: block;
}
</style>