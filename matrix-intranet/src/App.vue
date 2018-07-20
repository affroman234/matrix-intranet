<template>
  <div id="app">
    <login>
    </login>
    <ServerList>
    </ServerList>
  </div>
</template>

<script>
import Login from './components/Login'
import ServerList from './components/ServerList'

export default {
  name: 'App',
  components: {
    Login,
    ServerList
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
