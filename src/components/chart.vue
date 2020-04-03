<template>
  <div class="small">
    <pie-chart :chart-data="datacollection"></pie-chart>
  </div>
</template>

<script>
import pieChart from "./pieChart.js";

export default {
  props: ["dataset"],
  components: {
    pieChart
  },
  data() {
    return {
      datacollection: {}
    };
  },
  mounted() {
    this.fillData();
  },
  methods: {
    fillData() {
      this.datacollection = this.dataset;
    }
  }
};
</script>

<style>
.small {
  max-width: 300px;
}
</style>
