<template>
  <div>
    <div>
      <el-switch
        v-model="draggable"
        active-text="开启拖拽"
        inactive-text="关闭拖拽"
      >
      </el-switch>
      <el-button type="danger" round @click="batchDelete" size="mini"
        >批量删除</el-button
      >
      <el-button
        type="success"
        round
        v-if="draggable"
        @click="batchSave"
        size="mini"
        >保存</el-button
      >
    </div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
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
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
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
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
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
  components: {},
  props: {},
  data() {
    return {
      draggable: false,
      menus: [],
      dialogVisible: false,
      dialogType: "", //edit、add
      title: "",
      maxLevel: 0,
      pCid: [],
      updateNodes: [],
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  computed: {},
  watch: {},
  methods: {
    //获取树形分类菜单数据网络请求
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("数据：", data.data);
        this.menus = data.data;
      });
    },
    append(data) {
      this.category = {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      };
      this.dialogType = "add";
      this.title = "添加分类";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    edit(data) {
      this.title = "修改分类";
      this.dialogType = "edit";
      this.dialogVisible = true;
      //发送请求获取当前节点最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then((data) => {
        console.log("数据回显", data);
        var category = data.data.category;
        this.category.name = category.name;
        this.category.catId = category.catId;
        this.category.icon = category.icon;
        this.category.productUnit = category.productUnit;
        this.category.parentCid = category.parentCid;
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
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
            this.$message({
              type: "success",
              message: "菜单删除成功!",
            });
            //刷新出新的菜单
            this.getMenus();
            //设置需要默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
      console.log("remove", node, data);
    },
    //添加三级分类的方法
    addCategory() {
      console.log("提交三级菜单数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        //关闭对话框
        this.dialogVisible = false;
        this.$message({
          type: "success",
          message: "菜单保存成功!",
        });
        //刷新数据
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    //修改三级分类
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      console.log("修改菜单", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        //关闭对话框
        this.dialogVisible = false;
        this.$message({
          type: "success",
          message: "菜单修改成功!",
        });
        //刷新数据
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid];
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
    allowDrop(draggingNode, dropNode, type) {
      this.maxLevel = 0;
      //被拖动的当前结点以及所在的父节点总层数不能大于3
      console.log("allowDrop", draggingNode, dropNode, type);
      //被拖拽当前节点总层数
      let level = this.countNodeLevel(draggingNode);
      //当前正在拖拽的结点深度加上父节点的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log("深度", deep);
      if (type == "inner") {
        console.log(
          `this.maxLevel:${this.maxLevel};draggingNode.data.catLevel:${draggingNode.data.catLevel};dropNode.level:${dropNode.level}`
        );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      //找出所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单顺序修改成功!",
        });
        //刷新数据
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = this.pCid;
        //清空数据
        this.maxLevel = 0;
        this.updateNodes = [];
        this.pCid = [];
      });
    },
    //拖拽成功回调函数
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop", draggingNode, dropNode, dropType, ev);
      //1.当前结点最新父节点id
      let pCid = 0;
      let siblings = null;
      if (dropType == "inner") {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      } else {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      }
      this.pCid.push(pCid);
      //2.当前拖拽的结点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前正在拖拽的结点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            //当前结点的层级发生变化
            catLevel = siblings[i].level;
            //修改它子节点的层级
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
          });
        }
      }
      //3.当前拖拽结点的最新层级
      console.log("updateNodes", this.updateNodes);
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    //批量删除
    batchDelete() {
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素：", checkedNodes);
      let catIds = [];
      let names = [];
      catIds = checkedNodes.map((item) => item.catId);
      names = checkedNodes.map((item) => item.name);
      console.log("catIds", catIds);
      this.$confirm(`是否批量删除【${names}】菜单?`, "提示", {
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
            this.$message({
              type: "success",
              message: "菜单批量删除成功!",
            });
            //刷新出新的菜单
            this.getMenus();
          });
        })
        .catch(() => {});
    },
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {},
};
</script>

<style scoped>
</style>