<template>
  <nav>
    <router-link to="/contactos">Agenda</router-link> |
    <router-link to="/paises">Paises</router-link> |
    <router-link to="/ciudades">Ciudades</router-link> |
    <router-link to="/categorias">Categorías</router-link> |
    <router-link to="/empresas">Empresas</router-link>
  </nav>
  <router-view/>
</template>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
