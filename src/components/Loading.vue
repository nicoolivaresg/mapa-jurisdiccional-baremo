<template>
  <div class="card">
      <div class="card-body">
          <PulseLoader />
          <p class="h5">{{this.text}}</p>
      </div>
  </div>
</template>

<script>
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'

export default {
    components: {
        'PulseLoader': PulseLoader
    },
    props:{
        text: {
            default: 'Cargando...',
            type: String
        }
    }
}
</script>

<style>

</style>