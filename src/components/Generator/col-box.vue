<template>
  <div :key="key" class="col-box">
    <div v-if="pos == 0" class="top-left-cover-line" />
    <div v-if="pos == 0" class="bottom-left-cover-line" />
    <div v-if="pos == (total-1)" class="top-right-cover-line" />
    <div v-if="pos == (total-1)" class="bottom-right-cover-line" />
    <Node v-for="(item, index) in items" :key="index" :node="item" @addnode="addnode" @delNode="delNode(item)" @delConditionNode="delConditionNode" @addConditionFactor="addConditionFactor" />
  </div>
</template>
<script>
import {
  iteratorData,
  addNewNode,
  delNode,
  findIndex,
  deepClone,
} from "./process";
export default {
  name: "ColBox",
  props: {
    node: {
      type: Object,
      default: undefined,
    },
    total: {
      type: Number,
      default: 0,
    },
    pos: {
      type: Number,
      default: 0,
    },
  },
  data: () => ({
    items: [],
    // 用于强制刷新
    key: 0,
    node1: null,
  }),
  watch: {
    node: {
      handler(val) {
        // console.log(val)
        if (val) {
          this.getData(val);
          this.node1 = val;
        }
      },
      deep: true,
    },
  },
  mounted() {
    if (this.node) {
      this.getData(this.node);
      this.node1 = this.node;
    }
  },
  methods: {
    getData(data) {
      this.items = [];
      iteratorData(this.items, data);
    },
    addnode(node) {
      addNewNode(node, this.node1, this.items);
      this.key++;
    },
    delNode(node) {
      var index = findIndex(node.nodeId, this.items);
      /* console.log(this.node, "传过来的节点"); */
      this.items.splice(index, 1);
      /*  console.log(node, this.node1, this.items); */
      /* delNode(node, this.node1, this.items) */
      let data = deepClone(this.items);
      /* delete data[data.length - 1]["childNode"]; */
      if (data.length > 1) {
        data.shift();
      }
      if (data.length == 1) {
        data[0].prevId = this.node1.nodeId;
        this.$set(this.node1, "childNode", data[0]);
        this.key++;
        return;
      }
      for (let j = data.length - 1; j > -1; j--) {
        /* if (data[j] == 0 && !data[j - 1]) {
          this.$set(this.node1, "childNode", data[0]);
          this.$forceUpdate();
          this.key++;
          return;
        } */
        data[j - 1]["childNode"] = data[j];
        data[j].prevId = data[j - 1].nodeId;
        data.splice(j, 1);
      }
      this.$set(this.node1, "childNode", data[0]);
      this.$forceUpdate();
      this.key++;
    },
    getList(res, key) {
      let temp = {}; // 声明一个临时对象
      res[key] = temp; // 将返回的数据赋值给对象的属性key;
    },
    delConditionNode() {
      this.$emit("delConditionNode");
    },
    addConditionFactor(node) {
      this.$emit("addConditionFactor", node);
    },
  },
};
</script>
