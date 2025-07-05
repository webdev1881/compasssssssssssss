<template>
  <div id="app">
    <TableComponent />
    <TableComponent0 />
  </div>
</template>

<script>
import TableComponent from './components/TableComponent.vue'
import TableComponent0 from './components/TableComponent0.vue'

export default {
  name: 'App',
  components: {
    TableComponent,
    TableComponent0,
  }
}
</script>

<style>
#app {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
    'Helvetica Neue', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>