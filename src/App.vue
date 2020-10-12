<template>
  <div id="app">
    <course-table/>
  </div>

</template>

<script>
import courseTable from "./components/courseTable";

export default {
  name: 'App',
  components: {
    courseTable,
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
</style>
