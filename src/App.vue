<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  /* -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  color: #2c3e50;*/
  margin: 0 auto;  
  width:1100px;
  text-align: center;
}

body{
    background-color: #FFDCA8;
}

/* .log-form{
  margin-top:30px;
  margin-right:20px;
  margin-bottom: 80px;
  text-align: right;
} */
</style>
