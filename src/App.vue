<template>
  <div id="app">
    'SSSSSSSSS'
    <TableComponent />
    <Tdiv2 />
    <Tdiv />
    <Tt />
    <T />
    <TableComponent0 />
  </div>
</template>

<script>
import TableComponent from './components/TableComponent.vue'
import TableComponent0 from './components/TableComponent0.vue'
import T from './components/T.vue'
import Tdiv from './components/Tdiv.vue'
import Tdiv2 from './components/Tdiv2.vue'
import Tt from './components/Tt.vue'

export default {
  name: 'App',
  components: {
    TableComponent,
    TableComponent0,
    T,
    Tdiv,
    Tdiv2,
    Tt,
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