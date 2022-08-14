<template>
  <div class="d-flex flex-column h-100">
    <div   class="list-group overflow-auto flex-grow-1">
      <div id="graph" ref="graph"></div>
      {{nodes}}
    </div>
    <div class="d-flex pt-3">
      <div class="input-group">
        <input
        type="text"
        class="form-control first-item-radius"
        v-model="newNode"
        maxlength="50"
        placeholder="Enter node name..."
        @keyup.enter="addNode()"
        />
        <button class="btn btn-primary last-item-radius" @click="addNode()">
          <i class="bi bi-plus"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import * as _ from "lodash";
// import draggable from "vuedraggable";
import ForceGraph3D from '3d-force-graph';

export default {
  name: "GraphApp",
  // components: {
  //   draggable,
  // },
  props: {
    sync: Object,
  },
  data: () => ({
    nodes: [],
    links: [],
    newNode: "",
  }),
  computed: {
    checkedNodes() {
      return _.some(this.nodes, { done: true });
    },
  },
  watch: {
    checkedNodes(newVal) {
      const iconBtn = newVal
      ? [{ icon: "trash2", callback: this.clearDoneNodes }]
      : [];
      this.$emit("setHeaderButtons", iconBtn);
    },
  },
  created() {
    this.nodes = this.sync.get("nodes") || [];

    this.sync.observe((event) => {
      if (event.keysChanged.has("nodes")) {
        this.nodes = this.sync.get("nodes");
      }
    });



  },
  mounted(){
    // const N = 300;
    // const gData = {
    //   nodes: [...Array(N).keys()].map(i => ({ id: i })),
    //   links: [...Array(N).keys()]
    //   .filter(id => id)
    //   .map(id => ({
    //     source: id,
    //     target: Math.round(Math.random() * (id-1))
    //   }))
    // };
    let gData = {nodes: this.nodes, links: this.links}

    this.graph= ForceGraph3D()(this.$refs.graph)
    .graphData(gData)
    .width(800)
    .height(600)
  },
  methods: {
    draggableChange() {
      this.sync.set("nodes", this.nodes);
    },
    addNode() {
      if (this.newNode !== "") {
        this.nodes.push({ name: this.newNode, done: false });
        let nodesClean = this.nodes.map(function(item) {
          delete item.__threeObj;
          return item;
        });


        this.sync.set("nodes", nodesClean);
        this.newNode = "";
      }
    },
    toggleNodeState(node) {
      node.done = !node.done;
      this.sync.set("nodes", this.nodes);
    },
    clearDoneNodes() {
      this.nodes = _.filter(this.nodes, (t) => !t.done);
      this.sync.set("nodes", this.nodes);
    },
  },
};
</script>

<style scoped>
.node-done {
  text-decoration: line-through;
  color: gray;
}
</style>
