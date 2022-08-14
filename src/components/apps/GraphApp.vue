<template>
  <div class="d-flex flex-column h-100">
    <div   class="list-group overflow-auto flex-grow-1">
      <div id="graph" ref="graph"></div>
      <button @click="removeNodes">remove nodes</button>
      {{username}}
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
import { v4 as uuidv4 } from 'uuid';
// import draggable from "vuedraggable";
import ForceGraph3D from '3d-force-graph';

export default {
  name: "GraphApp",
  // components: {
  //   draggable,
  // },
  props: {
    sync: Object,
    username: String,
  },
  data: () => ({
    nodes: [],
    // links: [],
    newNode: "",
    synchro: null
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
    let app = this
    this.nodes = this.sync.get("nodes") || [];
    this.links = this.sync.get("links") || [];

    this.sync.observe((event) => {
      if (event.keysChanged.has("nodes")) {
        let nodes = this.sync.get("nodes");
        nodes.forEach((node) => {
          var index = app.nodes.findIndex(x => x.id==node.id);
          index === -1 ? app.nodes.push(node) : Object.assign(app.nodes[index], node)
        });

        // let links = this.sync.get("links")|| [];
        // links.forEach((link) => {
        //   var index = app.links.findIndex(x => x.id==link.id);
        //   index === -1 ? app.links.push(link) : Object.assign(app.links[index], link)
        // });
        //
        // console.log(this.links)
        this.graph.graphData({nodes: this.nodes, links: this.links})
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
    // .onEngineTick(() => this.onEngineTick())
    .width(800)
    .height(600)
    .nodeAutoColorBy('group')
    .onNodeDragEnd(node => {
      node.fx = node.x;
      node.fy = node.y;
      node.fz = node.z;
    });

    this.synchroToggle()

  },
  methods: {
    synchroToggle(){
      let app = this
      if (this.synchro == null){
        this.synchro = window.setInterval(function(){
          /// call your function here
          app.sync.set("nodes", app.cleanItems(app.nodes));
          // app.sync.set("links", app.links);
        }, 5000);
      }else{
        clearInterval(this.synchro)
        console.log(this.synchro)
      }
    },
    // onEngineTick(){
    //   console.log("tick")
    //   this.sync.set("nodes", this.cleanItems(this.nodes));
    //   this.sync.set("links", this.cleanItems(this.links));
    // },
    // draggableChange() {
    //   this.sync.set("nodes", this.nodes);
    // },
    addNode() {
      if (this.newNode !== "") {
        let newNode = { id: uuidv4(), name: this.newNode, done: false, group: this.username }
        this.nodes.push(newNode);
        if (this.nodes.length > 1){
          let t = Math.round(Math.random() * (this.nodes.length-1))
          let link = {source: newNode.id, target: this.nodes[t].id}
          this.links.push(link)
        }
        this.sync.set("nodes", this.cleanItems(this.nodes));
        // this.sync.set("links",this.links);
        this.newNode = "";
      }
    },
    cleanItems(items){
      return items.map(function(item) {
        delete item.__threeObj;
        return item;
      });
    },
    // toggleNodeState(node) {
    //   node.done = !node.done;
    //   this.sync.set("nodes", this.nodes);
    // },
    // clearDoneNodes() {
    //   this.nodes = _.filter(this.nodes, (t) => !t.done);
    //   this.sync.set("nodes", this.nodes);
    // },
    removeNodes() {
      this.nodes = []
      this.links = []
      // this.nodes = _.filter(this.nodes, (t) => !t.done);
      this.sync.set("nodes", this.nodes);
      this.sync.set("links", this.links);
    },
  },
};
</script>

/* <style scoped>
.node-done {
  text-decoration: line-through;
  color: gray;
}
</style> */
