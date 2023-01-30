<template>
  <div id="app">
    <LanguageSelector />
    <CellAutomatApp />
  </div>

  <div class="copyright">{{ $store.state.messages['copyright'] }}</div>
  <div>
    <a href="https://andrusbarabus.livejournal.com" target="_blank">{{
      $store.state.messages['myfotos']
    }}</a>
  </div>
</template>

<script>
import CellAutomatApp from './components/CellAutomatApp.vue';
import LanguageSelector from './components/LanguageSelector.vue';

export default {
  name: 'App',
  components: {
    CellAutomatApp,
    LanguageSelector,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 6px;
}

.copyright {
  font-size: 0.7em;
}
</style>
