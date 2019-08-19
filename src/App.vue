<template>
  <div id="app">
    <my-canvas></my-canvas>
<!--    <button @click="addLabel">+label</button>-->
  </div>
</template>

<script>
  // import MyCanvas from './components/my-canvas';
  import MyCanvas from './components/my-canvas-1';

  export default {
    name: 'app',
    components: {
      MyCanvas
    },
    methods: {
      addLabel() {
        alert('add label');
      }
    },
  };
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    /*text-align: center;*/
    /*color: #2c3e50;*/
    /*margin-top: 60px;*/
  }
</style>
