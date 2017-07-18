<template>
  <div style="postition: relative">
    <div class="action-container action-container__left">
      <b-button-group>
        <b-dropdown text="Layout">
          <b-dropdown-item @click.prevent="setLayout('concentric')">Concentric</b-dropdown-item>
          <b-dropdown-item @click.prevent="setLayout('grid')">Grid</b-dropdown-item>
          <b-dropdown-item @click.prevent="setLayout('hierarchical')">Hierarchy</b-dropdown-item>
        </b-dropdown>
        <b-btn v-b-modal.edit_data_modal>Edit Data</b-btn>
      </b-button-group>

      <b-modal id="edit_data_modal"
               title="Update Nodes and Edges"
               @ok="updateData">

        <form @submit.stop.prevent="updateData">
          <b-form-input type="number"
                        placeholder="Enter number of nodes"

                        v-model="numNodes"></b-form-input>
          <b-form-input type="number"
                        placeholder="Enter number of edges"

                        v-model="numEdges"></b-form-input>

        </form>

      </b-modal>

    </div>
    <div class="action-container action-container__right">
      <b-button-group vertical>
        <b-button variant="default"
                  @click="zoomReset()"
                  title="Zoom Reset">
          <svgicon icon="home"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
        <div class="divider"></div>
        <b-button variant="default"
                  @click="zoomIn()"
                  title="Zoom In">
          <svgicon icon="plus"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
        <b-button variant="default"
                  @click="zoomOut()"
                  title="Zoom In">
          <svgicon icon="minus"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
        <div class="divider"></div>
        <b-button variant="default"
                  @click="locateRandomNode()"
                  title="Locaterandom node">
          <svgicon icon="near-me"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
        <div class="divider"></div>
        <b-button variant="default"
                  @click="rotateLeft()"
                  title="Rotate Left">
          <svgicon icon="rotate-left"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
        <b-button variant="default"
                  @click="rotateRight()"
                  title="Rotate Right">
          <svgicon icon="rotate-right"
                   width="22"
                   height="18">
          </svgicon>
        </b-button>
      </b-button-group>
    </div>
    <div id="graph-container"></div>
    <div v-if="graphData.nodes"
         class="graph-info">
      <div v-if="graphData.nodes.length">Nodes: {{graphData.nodes.length}}</div>
      <div v-if="graphData.edges.length">Edges: {{graphData.edges.length}}</div>
    </div>

  </div>
</template>

<script>
import * as Ogma from '../../vendors/ogma.min';
import * as dagre from '../../node_modules/dagre/dist/dagre'; // eslint-disable-line no-unused-vars

import '../../compiled-icons/home';
import '../../compiled-icons/near-me';
import '../../compiled-icons/plus';
import '../../compiled-icons/minus';
import '../../compiled-icons/rotate-left';
import '../../compiled-icons/rotate-right';

window.dagre = dagre;

export default {
  data() {
    return {
      numNodes: 50,
      numEdges: 50,
      graphData: {},
      selectedNodes: []
    };
  },
  mounted() {
    this.graphData = this.randomGraph(this.numNodes, this.numEdges);
    this.ogma = new Ogma({
      graph: this.graphData,
      container: 'graph-container',
      plugins: ['dagre']
    });

    // setInterval(() => {
    //   ogma.graph.addNode({ id: Math.random() });
    // }, 3000);

    // lasso selection on shift press
    this.ogma.events.bind('keyboard.keyDown', (key) => {
      if (key === 'shift') {
        if (this.ogma.keyboard.isPressed('shift')) {
          this.ogma.lasso.start();
        } else {
          this.ogma.lasso.stop();
        }
      }
    });

    this.ogma.events.bind({
      'selection.nodesSelected': () => {
        this.selectedNodes = this.ogma.selection.nodes();
      }
    });
  },
  methods: {
    updateData() {
      this.ogma.graph.clear();
      this.graphData = this.randomGraph(this.numNodes, this.numEdges);
      this.ogma.graph.addNodes(this.graphData.nodes);
      this.ogma.graph.addEdges(this.graphData.edges);
    },

    randomGraph(_nodes, _edges) {
      const g = { nodes: [], edges: [] };

      for (let i = 0; i < _nodes; i++) {
        g.nodes.push({
          id: `n ${i}`,
          x: Math.random() * 100,
          y: Math.random() * 100
        });
      }

      for (let i = 0; i < _edges; i++) {
        g.edges.push({
          id: `e ${i}`,
          source: `n ${(Math.random() * _nodes | 0)}`,
          target: `n ${(Math.random() * _nodes | 0)}`
        });
      }

      return g;
    },

    setLayout(layout) {
      switch (layout.toLowerCase()) {
        case 'concentric':
          if (this.selectedNodes && this.selectedNodes.length === 1) {
            this.ogma.layouts.start('concentric', {
              centralNode: this.selectedNodes[0].id,
              allowOverlap: false
            }, { onEnd: this.zoomReset });
          } else {
            alert('You must only one node'); // eslint-disable-line no-alert
          }
          break;

        case 'grid':
          this.ogma.layouts.start('grid', {
            // nbRows: 8,
            // nbCols: 8,
            allowOverlap: false
            // sortBy: 'size' // or any node property
          }, { onEnd: this.zoomReset });
          break;

        case 'hierarchical':
          this.ogma.dagre.start({
            directed: true, // Take edge direction into account
            rankdir: 'TB',  // Direction for rank nodes. Can be TB, BT, LR, or RL,
            // where T = top, B = bottom, L = left, and R = right.
            duration: 300,  // Duration of the animation
            nodesep: 30,     // Number of pixels that separate nodes horizontally in the layout.
            ranksep: 40     // Number of pixels between each rank in the layout.
          }, { onEnd: this.zoomReset });
          break;

        // no default
      }
    },
    zoomReset() {
      const angle = this.ogma.camera.angle;
      let rotation;
      if (angle > 180) {
        rotation = 360 - angle;
      } else {
        rotation = -1 * angle;
      }
      this.ogma.camera.rotate(rotation, {  // angle in radian
        duration: 300
        // callback() { console.log('rotation done'); }
      });
      this.ogma.locate.center({
        easing: 'linear',
        duration: 300
        // padding: { right: 100 }
      });
    },

    zoomIn() {
      this.ogma.camera.zoomIn({
        duration: 200
        // callback() { console.log('zoom done'); }
      });
    },

    zoomOut() {
      this.ogma.camera.zoomOut({
        duration: 200
        // callback() { console.log('zoom done'); }
      });
    },

    locateRandomNode() {
      this.ogma.locate.nodes('n 0', { duration: 600 });
      this.ogma.selection.addNodes(['n 0']);
    },

    locateRandomEdge() {
      this.ogma.locate.edges(['e 0'], { duration: 600 });
    },

    rotateLeft() {
      this.ogma.camera.rotate(Math.PI / 2, {  // angle in radian
        duration: 600
        // callback() { console.log('rotation done'); }
      });
    },

    rotateRight() {
      this.ogma.camera.rotate(-Math.PI / 2, {  // angle in radian
        duration: 600
        // callback() { console.log('rotation done'); }
      });
    },

    addNodes() {
      // const nodes = [];
      this.ogma.graph.addNodes({ id: 'n0' });
    }
  }
};
</script>

<style>
#graph-container {
  /* top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  position: absolute;
  margin: 0;
  overflow: hidden; */
  margin: 0;
  overflow: hidden;
  height: calc(100vh - 50px);
  width: 100%;
}

.action-container {
  position: absolute;
  top: 59px;
  /* border: 1px solid silver; */
  background-color: #ffffff;
  box-shadow: 0 1px 6px rgba(0, 0, 0, 0.16),
  0 1px 6px rgba(0, 0, 0, 0.23);
}

.action-container__left {
  left: 8px;
}

.action-container__right {
  right: 8px;
}

.divider {
  border-bottom: 1px solid silver;
}

.btn-squared {
  border-radius: 0 !important;
}

.btn-group-vertical>.btn.btn-squared {
  border-radius: 0;
}

.graph-info {
  position: absolute;
  bottom: -6px;
  left: 0px;
  background-color: black;
  color: white;
  padding: 0px 4px;
}
</style>
