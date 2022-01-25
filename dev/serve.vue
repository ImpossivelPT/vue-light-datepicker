<script>
import Vue from 'vue';
import VueLightDatepicker from '@/vue-light-datepicker.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueLightDatepicker
  }
});
</script>

<template>
  <div id="app">
    <vue-light-datepicker roundCorners/>
  </div>
</template>
