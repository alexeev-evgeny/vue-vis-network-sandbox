<template>
	<div id="app">
		<!-- <img alt="Vue logo" src="./assets/logo.png"> -->
		<!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->

		<table style="width: 100%; table-layout: fixed;">
			<tr>
				<td style="vertical-align: top;">
					<div 
						id="mynetwork" 
						ref="myNetwork" 
					></div>
					<!-- <button @click="createNewNode">CREATE NEW NODE</button> -->
				</td>
				<td style="vertical-align: top;">

					<div
						style="text-align: left; background-color: lightgreen; width: 300px;"
					>
						<span>New node</span>
						<br/>
						<input 
							v-model="newNode.label"
							type="text" 
							placeholder="Label" 
						/>
						<br/>
						<input 
							v-model="newNode.edgeFrom"
							type="text" 
							placeholder="Edge from" 
						/>
						<br/>
						<input 
							v-model="newNode.edgeTo"
							type="text" 
							placeholder="Edge to" 
						/>
						<br/>
						<button @click="createNewNode">CREATE</button>
					</div>

					<br/>
					<br/>
					<br/>

					<div style="text-align: left; background-color: lightgreen; width: 300px;">
						<span>Remove node</span>
						<br/>
						<input 
							v-model="nodeIdToRemove"
							type="text"
							placeholder="NodeId" 
						/>
						<br/>
						<button @click="removeNode">REMOVE</button>
					</div>

				</td>
			</tr>
		</table>


		<table style="width: 100%">
			<tr>
				<td style="vertical-align: top;">
					<pre style="text-align: left">nodes: {{nodes}}</pre>
				</td>
				<td style="vertical-align: top;">
					<pre style="text-align: left">edges: {{edges}}</pre>
				</td>
			</tr>
		</table>

	</div>
</template>

<script>
import * as vis from 'vis-network';
// import HelloWorld from './components/HelloWorld.vue'

const DEFAULT_NODE_PARAMS = {
	shape: 'box',
	margin: 10,
}

const DEFAULT_NODES = [
	{ id: 1, label: "Node 1", x: 0, y: 0 },
	{ id: 2, label: "Node 2", x: 200, y: 0 },
	{ id: 3, label: "Node 3", x: 400, y: 0 },
	{ id: 4, label: "Node 4", x: 600, y: 0 },
	{ id: 5, label: "Node 5", x: 800, y: 0 },
].map((i) => ({
	...i,
	...DEFAULT_NODE_PARAMS
}));

const DEFAULT_EDGE_PARAMS = {
	arrows: 'to',
	physics: false,
}

const DEFAULT_EDGES = [
	{ from: 1, to: 2 },
	{ from: 2, to: 3 },
	{ from: 3, to: 4 },
	{ from: 4, to: 5 },
].map(i => ({
	...i,
	...DEFAULT_EDGE_PARAMS
}));

const DEFAULT_OPTIONS = {
	locale: 'ru',
	physics: {
		enabled: false,
	},
	layout: {
		randomSeed: 5
	}
};

export default {
	name: 'App',
	components: {
		// HelloWorld
	},
	data() {
		return {
			selectedNode: null,
			nodeIdToRemove: null,

			network: null,
			nodes: DEFAULT_NODES,
			edges: DEFAULT_EDGES,
			options: DEFAULT_OPTIONS,

			newNode: {
				label: '',
				edgeFrom: null,
				edgeTo: null,
				...DEFAULT_NODE_PARAMS
			}
		}
	},

	mounted() {
		console.warn({vis});

		this.fetchState();
		this.updateNetwork();

		console.warn({ network: this.network });
	},
	methods: {
		updateNetwork() {
			const data = { 
				nodes: this.nodes, 
				edges: this.edges 
			};
			
			console.error('UPDATE NETWORK', JSON.stringify(data, null, 4));

			this.network = new vis.Network(
				this.$refs.myNetwork,
				data,
				this.options
			);
			this.network.on('selectNode', this.onNodeClick);
			this.network.on('dragEnd', this.onNodeDragEnd);
		},

		createNewNode() {
			const nodeId = this.nodes.length + 1;
			this.nodes.push({
				id: nodeId,
				label: this.newNode.label || `Node ${nodeId}`,
			})

			const edgeFrom = Number(this.newNode.edgeFrom);
			const edgeTo = Number(this.newNode.edgeTo);

			if (edgeFrom) {
				this.edges.push({
					from: edgeFrom,
					to: nodeId,
					...DEFAULT_EDGE_PARAMS
				})
			}

			if (edgeTo) {
				this.edges.push({
					from: nodeId,
					to: edgeTo,
					...DEFAULT_EDGE_PARAMS
				})
			}

			console.warn({ nodeId }, this.nodes);

			this.saveState();
			this.updateNetwork();
		},

		removeNode() {
			console.error('REMOVE NODE', { id: this.nodeIdToRemove });
			const nodeId = Number(this.nodeIdToRemove);

			this.nodes = this.nodes.filter(node => node.id !== nodeId);
			this.edges = this.edges.filter(edge => edge.from !== nodeId && edge.to !== nodeId);

			this.saveState();
			this.updateNetwork();
		},

		onNodeClick(event) {
			console.error('CLICK', event);

			console.warn(event.pointer.canvas);

			const nodeId = event.nodes[0];

			this.selectedNode = this.nodes.find(node => node.id === nodeId);

			console.warn('SELECTED NODE', JSON.stringify(this.selectedNode))
		},

		onNodeDragEnd(event) {
			console.error('DRAG END', event);
			
			const id = event.nodes[0];
			const position = event.pointer.canvas;
			
			console.warn({ id, position: JSON.stringify(position) });

			this.nodes = this.nodes.map(node => {
				if (node.id === id) {
					return { ...node, ...position }
				}
				return node;
			});

			this.saveState();
		},

		saveState() {
			const state = JSON.stringify({
				nodes: this.nodes,
				edges: this.edges,
				options: this.options,
			});
			window.localStorage.setItem('my-network', state);

			console.warn('SAVE STATE', { state });
		},

		fetchState() {
			const localData = window.localStorage.getItem('my-network');
			
			console.error('FETCH STATE', {localData});

			if (!localData) {
				return;
			}

			const { nodes, edges, options } = JSON.parse(localData);
			
			console.error('FETCH STATE', { nodes, edges, options });

			this.nodes = nodes;
			this.edges = edges;
			this.options = options;
		},
	}
}
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}

#mynetwork {
	width: 100%;
	height: 1000px;
	background-color: pink;
	/* margin: 0 auto; */
}
</style>
