<template>
  <div id="app">
    <!--<img alt="Vue logo" src="./assets/logo.png">-->
    <!--<HelloWorld msg="Welcome to Your Vue.js App"/>-->
    <my-canvas></my-canvas>
    <button @click="addLabel">+label</button>
  </div>
</template>

<script>
  import HelloWorld from './components/HelloWorld.vue';
  import MyCanvas from './components/my-canvas';

  export default {
    name: 'app',
    components: {
      // HelloWorld,
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
