<template>
  <div id="app">

    <b-navbar toggleable
              type="inverse"
              variant="inverse">

      <b-link class="navbar-brand"
              to="#">
        <span>Ogma Test</span>
      </b-link>

      <b-nav is-nav-bar>
        <b-nav-item to="/"
                    exact>Home</b-nav-item>
        <!-- These links need to defer to the browser navigator -->
        <!-- ignore the url prefixing... it's for a reverse proxy, not a vue-router thing -->
        <b-nav-item to="/graph">Graph</b-nav-item>
      </b-nav>
    </b-navbar>

    <main style="height: calc(100vh - 72px); margin-top:0">
      <router-view></router-view>
    </main>
  </div>
</template>

<script>
export default {
  name: 'app'
};
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

main {
  text-align: center;
  margin-top: 40px;
}

header {
  margin: 0;
  height: 56px;
  padding: 0 16px 0 24px;
  background-color: #35495E;
  color: #ffffff;
}

header span {
  display: block;
  position: relative;
  font-size: 20px;
  line-height: 1;
  letter-spacing: .02em;
  font-weight: 400;
  box-sizing: border-box;
  padding-top: 16px;
}

.router-link-active {
  color: #555;
  background-color: #e7e7e7;
}
</style>
