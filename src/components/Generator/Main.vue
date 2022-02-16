<template>
  <div style="width:100%;height:100%;">
    <div class="fd-nav">
      <div class="fd-nav-left">
        <div @click="close" class="fd-nav-back">
          <i aria-label="icon: left" class="anticon anticon-left"><svg viewBox="64 64 896 896" focusable="false" class="" data-icon="left" width="1em" height="1em" fill="currentColor" aria-hidden="true">
              <path d="M724 218.3V141c0-6.7-7.7-10.4-12.9-6.3L260.3 486.8a31.86 31.86 0 0 0 0 50.3l450.8 352.1c5.3 4.1 12.9.4 12.9-6.3v-77.3c0-4.9-2.3-9.6-6.1-12.6l-360-281 360-281.1c3.8-3 6.1-7.7 6.1-12.6z" />
            </svg></i>
        </div>
        <div class="fd-nav-title">
          {{ data1.title }}
        </div>
      </div>
      <div class="fd-nav-center">
        <div class="fd-nav-container">
          <div class="fd-nav-item">
            <span>流程设计</span>
          </div>
        </div>
      </div>
      <div class="fd-nav-right">
        <!-- <button type="button" class="ant-btn button-preview" @click="preview">
          <span>预 览</span>
        </button> -->
        <button type="button" v-if="disabled" class="ant-btn button-preview" @click="save">
          <span>{{title}}</span>
        </button>
      </div>
    </div>
    <div class="fd-nav-content">
      <div class="dingflow-design">
        <!-- <div v-if="viewModal" class="dialog-modal-mask"></div> -->
        <AModal :dialog.sync="viewModal" style="z-index:2000;">
          <pre style="font-family: Monaco,Menlo,Consolas,Bitstream Vera Sans Mono,monospace;font-size: 14px;">{{ JSON.stringify(data1.node, null, 4) }}</pre>
        </AModal>
        <div class="ie-polyfill-container">
          <div id="box-scale" :key="key" class="box-scale" style="transform: scale(1); transform-origin: 50% 0px 0px;">
            <Node v-for="(item, index) in items" :key="index" :node="item" @addnode="addnode" @delNode="delNode(item)" />
            <EndNode />
            <ErrorsModal :dialog.sync="errorsModal" :data="errors" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import AModal from "./../AModal/AModal";
import EndNode from "./end-node";
import ErrorsModal from "./errors-modal";
import {
  iteratorData,
  addNewNode,
  checkData,
  findIndex,
  deepClone,
} from "./process";
export default {
  name: "WorkflowUi",
  components: {
    EndNode,
    AModal,
    ErrorsModal,
  },
  props: {
    title: {
      type: String,
      default: "发布",
    },
    data: {
      type: Object,
      default: undefined,
    },
    disabled: {
      type: Boolean,
      default: true,
    },
  },
  data: () => ({
    items: [],
    key: 0,
    errorsModal: false,
    errors: [],
    viewModal: false,
    data1: {
      title: "请假",
      node: {
        name: "发起人",
        type: "start",
        nodeId: "sid-startevent",
        childNode: {},
      },
    },
    newarr: [],
  }),
  watch: {
    data: {
      handler(val) {
        this.data1 = val;
      },
      deep: true,
    },
  },
  mounted() {
    if (this.data && this.data.node) {
      this.data1 = this.data;
    }
    if (!this.data1.node) {
      this.initialNode();
    }
    this.iteratorData(this.data1.node);
  },
  methods: {
    close() {
      var errors = checkData(this.data1.node);
      if (errors.length > 0) {
        this.errorsModal = true;
        this.errors = errors;
        return;
      }
      this.$emit("close");
    },
    initialNode() {
      this.data1.node = {
        name: "发起人",
        type: "start",
        nodeId: "sid-startevent",
      };
    },
    iteratorData(data) {
      this.items = [];
      iteratorData(this.items, data);
    },
    addnode(node) {
      addNewNode(node, this.data1.node, this.items);
      this.key++;
    },
    delenode(nodeDel, node, arr) {
      // 从遍历后数组中删除节点
      var index = findIndex(nodeDel.nodeId, arr);
      arr.splice(index, 1);
      if (arr.length == 0) {
        let item = [
          {
            name: "发起人",
            type: "start",
            nodeId: "sid-startevent",
            childNode: {},
          },
        ];
        this.$set(this, "items", item);
        this.$set(this.data1.node, "childNode", {});
        this.$forceUpdate();
        return;
      }
      let data = deepClone(arr);
      delete data[data.length - 1]["childNode"];
      for (let j = arr.length - 1; j > -1; j--) {
        if (!data[j - 1]) {
          arr.unshift({
            name: "发起人",
            type: "start",
            nodeId: "sid-startevent",
            childNode: {},
          });
          this.$set(this, "items", arr);
          this.$set(this.data1.node, "childNode", data[0]);
          this.$forceUpdate();
          return;
        }
        data[j - 1]["childNode"] = data[j];
        data[j].prevId = data[j - 1].nodeId;
        data.splice(j, 1);
      }
      arr.unshift({
        name: "发起人",
        type: "start",
        nodeId: "sid-startevent",
        childNode: {},
      });
      this.$set(this, "items", arr);
      this.$set(this.data1.node, "childNode", data[0]);
      this.$forceUpdate();
    },
    deleteNode(nodeDel, node) {
      var temp = node;
      // 找到删除节点的父节点
      while (temp.childNode) {
        if (temp.nodeId == nodeDel.prevId) {
          // 将删除节点的子节点指向父节点
          if (nodeDel.childNode == null) {
            temp.childNode = null;
            return;
          }
          nodeDel.childNode.prevId = temp.nodeId;
          temp.childNode = nodeDel.childNode;
          return;
        }
        // 循环结束
        if (temp.childNode != null) temp = temp.childNode;
      }
    },
    delNode(node) {
      const data = deepClone(this.data1);
      this.newarr = [];
      this.tree2(data.node);
      this.delenode(node, this.data1.node, this.newarr);
      this.key++;
    },
    tree2(obj) {
      for (let i in obj) {
        if (i == "childNode" && obj[i]) {
          this.newarr.push(obj[i]);
          this.tree2(obj[i]);
          if (obj[i]) {
            obj[i] = {};
          }
        }
      }
    },
    save() {
      var errors = checkData(this.data1.node);
      if (errors.length > 0) {
        this.errorsModal = true;
        this.errors = errors;
        return;
      }
      this.$emit("ok", this.data1);
    },
    preview() {
      var errors = checkData(this.data1.node);
      if (errors.length > 0) {
        this.errorsModal = true;
        this.errors = errors;
        return;
      }
      this.$set(this.data, "viewModal", true);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style  lang="scss">
@media screen and (max-width: 500px) {
  .fd-nav {
    position: relative;
    z-index: 0;
    width: 100%;
    height: 60px;
    font-size: 14px;
    color: #fff;
    background: #3296fa;
    display: flex;
    align-items: center;
    .fd-nav-center {
      width: 150px !important;
    }
  }
  .ant-modal {
    width: 100% !important;
  }
}
.fd-nav-content {
  position: relative;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  overflow-x: auto;
  overflow-y: auto !important;
  height: 100%;
}
</style>
