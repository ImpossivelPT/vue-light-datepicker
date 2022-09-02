<script>
import Vue from 'vue';
import VueLightDatepicker from '@/vue-light-datepicker.vue';

export default Vue.extend({
  name: 'ServeDev',
  data() {
    return {
      myDate: [],
      dateExternal: null,
    }
  },
  components: {
    VueLightDatepicker
  }
});
</script>

<template>
  <div id="app">
    <vue-light-datepicker v-model="myDate" language="en" :allowSelectSameDay="false" roundCorners range showFromTo="to" date="" />
    <button @click="dateExternal='27/05/2022'">Set value extenally</button>
  </div>
</template>
