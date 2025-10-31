<template>
  <div id="app">
    <h1>Hello Vue 2!</h1>
    <div id="mxGraphContainer" style="width: 800px; height: 600px;"></div>
  </div>
</template>

<script>
import mxgraph from 'mxgraph';

export default {
  name: 'App',
  mounted() {
    const container = document.getElementById('mxGraphContainer');
    const graph = new mxgraph.mxGraph(container);
    // Additional mxGraph setup here
  }
};
</script>

<style>
#mxGraphContainer {
  border: 1px solid black;
}
</style>