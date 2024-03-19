<template>
  <div id="app">
   <car></car>
  </div>
</template>

<script>
import car from '@/components/car.vue';

export default {
  name: 'App',
  components: {
    car
  }
}
</script>

<style>
*{
    margin: 0px;
    padding: 0px;
}
</style>
