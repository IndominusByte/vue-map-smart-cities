<template>
  <div id="app">
    <google-map></google-map>
  </div>
</template>

<script>
import GoogleMap from './components/GoogleMap.vue'

export default {
  name: 'App',
  components: {
    GoogleMap
  }
}
</script>
