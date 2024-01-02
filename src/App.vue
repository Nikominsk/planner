<template>
  <div id="nav">
    <router-link to="/">Schedule</router-link> |
    <router-link to="/about">PDF Preview</router-link>
  </div>
  <router-view />
</template>

<style>

html {
  print-color-adjust: exact;
  -webkit-print-color-adjust: exact;
}


#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
