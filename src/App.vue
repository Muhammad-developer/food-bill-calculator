<template>
  <v-app>
    <v-main>
      <BillCalculator />
    </v-main>
  </v-app>
</template>

<script>
import BillCalculator from './components/BillCalculator.vue';

export default {
  components: {
    BillCalculator
  }
};
</script>
