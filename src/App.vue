<template>
  <div id="app">
      <portal-target name="semantic-ui-vue">
      </portal-target>
      <div class="flex h-16 text-white bg-black items-center text-center"> 
      <div class="w-1/6 text-4xl">DaViD</div>
      <div id="nav" class="align-middle text-left w-full">
        <router-link to="/">Home</router-link> |
        <router-link to="/map">Map</router-link> |
        <router-link to="/about">About</router-link>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.ml-100 {
  margin-left: 200px;
}
</style>
