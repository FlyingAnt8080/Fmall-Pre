<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
    >
    </el-tree>
  </div>
</template>

<script>
export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    //这里存数据
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  //计算属性
  computed: {},
  //监控data中数据变化
  watch: {},
  //方法
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
    nodeClick(data, node, component) {
      console.log("子组件的结点被点击", data, node, component);
      //相父组件发送事件
      this.$emit("tree-node-click", data, node, component);
    },
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //声明周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁之后
  activated() {}, //缓存keep-alive
};
</script>

<style scoped>
</style>