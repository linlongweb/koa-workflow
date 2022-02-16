<template>
  <div id="app">
    <Main :data.sync="data" @close="close" @ok="ok" :roledata.sync="roledata" />
  </div>
</template>

<script>
import Main from "./components/Generator/Main";
export default {
  name: "App",
  components: {
    Main,
  },
  props: {
    data: {
      type: Object,
      default: undefined,
    },
    roledata: {
      type: Array,
      default: undefined,
    },
  },
  data() {
    return {};
  },
  methods: {
    ok(data) {
      this.$emit("update:data", data);
    },
    close() {
      this.$emit("close");
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
