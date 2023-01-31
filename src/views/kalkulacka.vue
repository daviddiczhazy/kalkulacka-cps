<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/calc.png" width="230" />
    <vstupnyFormular msg="Kalkulačka na výpočet cestovného poistenia" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import vstupnyFormular from '@/components/vstupnyFormular.vue' // @ is an alias to /src

export default defineComponent({
  name: 'kalkulackaCp',
  components: {
    vstupnyFormular,
  },
})
</script>
