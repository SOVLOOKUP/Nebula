<script>
  export let dataURL = "";
  import ForceGraph3D from "3d-force-graph";
  import Loading from "../loading/index.svelte";
  let threedGraph;
  const NODES = 50;
  const GROUPS = 12;

  const fakeData = {
    nodes: [...Array(NODES).keys()].map((i) => ({
      id: i,
      group: Math.ceil(Math.random() * GROUPS),
      name: "节点名称",
      img: "",
    })),
    links: [...Array(NODES).keys()]
      .filter((id) => id)
      .map((id) => ({
        source: id,
        target: Math.round(Math.random() * (id - 1)),
        value: Math.round(Math.random() * (id - 1)),
        group: Math.ceil(Math.random() * GROUPS),
      })),
  };

  // fetch data
  async function fetchData(dataURL) {
    async function getRemoteData() {
      const res = await fetch(dataURL);
      if (res.ok) {
        return res.json();
      } else {
        throw new Error(res.json());
      }
    }

    return dataURL === "" ? fakeData : getRemoteData();
  }
  // display graph
  async function displayGraph(gData) {
    const highlightNodes = new Set();
    const highlightLinks = new Set();
    let hoverNode = null;
    function updateHighlight() {
      // trigger update of highlighted objects in scene
      Graph.nodeColor(Graph.nodeColor())
        .linkWidth(Graph.linkWidth())
        .linkDirectionalParticles(Graph.linkDirectionalParticles());
    }
    gData.links.forEach((link) => {
      const a = gData.nodes[link.source];
      const b = gData.nodes[link.target];
      !a.neighbors && (a.neighbors = []);
      !b.neighbors && (b.neighbors = []);
      a.neighbors.push(b);
      b.neighbors.push(a);

      !a.links && (a.links = []);
      !b.links && (b.links = []);
      a.links.push(link);
      b.links.push(link);
    });

    const Graph = ForceGraph3D()(threedGraph)
      .graphData(gData)
      //   show navinfo
      //   .showNavInfo(false)
      .linkWidth((link) => (highlightLinks.has(link) ? 4 : 1))
      .nodeAutoColorBy("group")
      .linkAutoColorBy((link) => link.group)
      .linkDirectionalParticles("value")
      .linkDirectionalParticleSpeed((d) => d.value * 0.001)
      .linkDirectionalParticleWidth(1)
      .linkDirectionalArrowLength(3.5)
      .linkDirectionalArrowRelPos(1)
      .linkCurvature(0.25)
      .onNodeHover((node) => {
        // no state change
        if ((!node && !highlightNodes.size) || (node && hoverNode === node))
          return;

        highlightNodes.clear();
        highlightLinks.clear();
        if (node) {
          highlightNodes.add(node);
          node.neighbors.forEach((neighbor) => highlightNodes.add(neighbor));
          node.links.forEach((link) => highlightLinks.add(link));
        }

        hoverNode = node || null;

        updateHighlight();
      })
      .onLinkHover((link) => {
        highlightNodes.clear();
        highlightLinks.clear();

        if (link) {
          highlightLinks.add(link);
          highlightNodes.add(link.source);
          highlightNodes.add(link.target);
        }

        updateHighlight();
      })
      .onNodeClick((node) => {
        // Aim at node from outside it
        const distance = 65;
        const distRatio = 1 + distance / Math.hypot(node.x, node.y, node.z);

        Graph.cameraPosition(
          {
            x: node.x * distRatio,
            y: node.y * distRatio,
            z: node.z * distRatio,
          }, // new position
          node, // lookAt ({ x, y, z })
          3000 // ms transition duration
        );
      });
  }
</script>

{#await fetchData(dataURL)}
  <Loading notice="获取数据中" />
{:then gData}
  {#await displayGraph(gData) then ok}
    <Loading notice="渲染中" />
  {/await}
{/await}
<div class="threed" bind:this={threedGraph} />

<style>
  .threed {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }
</style>
