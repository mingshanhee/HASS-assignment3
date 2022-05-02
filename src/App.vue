<template>
  <div v-if="data == null">Loading...</div>
  <div class="container" v-else>
    <div class="mb-5">
      Hypothesis/Task Abstraction: I am interested to find out

      <br />
      <br />
      <div style="text-align: left">
        (1) What are the popular town areas for "4-ROOM" resale flat
        transactions in 2021?

        <br />
        (2) What is the median price for the "4-ROOM" resale flat in each town
        area in 2021 (in square meter)?

        <br />
        (3) Does the median price for "4-ROOM" resale flat have a relationship
        with the number of transactions in the area?
      </div>
    </div>
    <div class="row mb-3">
      <div class="col-sm-2">
        <span><b>Select region:</b></span>
      </div>
      <div class="col-sm-10">
        <select
          class="form-select form-select-sm"
          v-model="selectFlatType"
          @change="onSelectChange"
        >
          <option
            v-for="ft in flatTypes"
            v-bind:key="ft"
            v-bind:value="ft"
            :selected="ft == selectFlatType"
          >
            {{ ft }}
          </option>
        </select>
      </div>
    </div>

    <div></div>

    <br />

    <div class="container" v-if="flatTypeData.length > null">
      <span
        >There are <b>{{ this.flatTypeData.length }}</b> transaction records in
        this region from Jan 2021 to Dec 2021
      </span>
      <br />
      <br />
      <div style="text-align: right">
        <span style="margin: 8px">
          Display:
          <div class="btn-group" role="group" aria-label="Basic example">
            <button
              type="button"
              class="btn"
              :class="{
                'btn-info': this.chart == 'barchart',
                'btn-outline-info': this.chart == 'boxplot',
              }"
              @click="setChart('barchart')"
            >
              # Transactions
            </button>
            <button
              type="button"
              class="btn"
              :class="{
                'btn-info': this.chart == 'boxplot',
                'btn-outline-info': this.chart == 'barchart',
              }"
              @click="setChart('boxplot')"
            >
              Price Per Square Meter
            </button>
          </div>
        </span>
        <span>
          Order By (# Transaction Records, Double-Click due to some buggy race
          condition):
          <div class="btn-group" role="group" aria-label="Basic example">
            <button
              type="button"
              class="btn"
              :class="{
                'btn-info': this.ascending,
                'btn-outline-info': !this.ascending,
              }"
              @click="setOrder(true)"
            >
              Ascending
            </button>
            <button
              type="button"
              class="btn"
              :class="{
                'btn-info': !this.ascending,
                'btn-outline-info': this.ascending,
              }"
              @click="setOrder(false)"
            >
              Descending
            </button>
          </div>
        </span>
      </div>
      <bar-chart
        ref="barchart"
        v-bind:data="flatTypeData"
        v-bind:towns="towns"
        v-bind:ascending="ascending"
        v-if="chart == 'barchart'"
      ></bar-chart>
      <box-plot
        ref="boxplot"
        v-bind:data="flatTypeData"
        v-bind:towns="towns"
        v-bind:ascending="ascending"
        v-if="chart == 'boxplot'"
      ></box-plot>

      <div class="mt-3" style="text-align: left">
        <span style="color: red"
          >The answers to the task abstractions are in red</span
        >
        <br />

        (1) What are the popular town areas for "4-ROOM" resale flat
        transactions in 2021?
        <br />
        -
        <span style="color: red"
          >The top 3 areas for resale flats transactions are in Punggol,
          Sengkang and Tampines</span
        >

        <br />
        (2) What is the median price for the "4-ROOM" resale flat in each town
        area in 2021 (in square meter)?
        <br />
        -
        <span style="color: red"
          >The median price for "4-ROOM" resale flat ranges between 4,750 to
          5,250 in top 3 town areas with most number of transactions</span
        >

        <br />
        (3) Does the median price for "4-ROOM" resale flat have a relationship
        with the number of transactions in the area?
        <br />
        -
        <span style="color: red"
          >It seems like there is a relationship where the median price are
          relatively lower in the top-5 town areas. Comparatively, the median
          price in the Central town area is significantly higher and there are
          far fewer transactions</span
        >
      </div>
    </div>
  </div>
</template>

<script>
import records from "../sg_resale_flats_2021.json";
// import StackBarPlot from "./StackBarPlot.vue";
import BoxPlot from "./BoxPlot.vue";
import BarChart from "./BarChart.vue";

export default {
  components: { BoxPlot, BarChart },
  name: "App",
  data() {
    return {
      data: records,
      item: records[0],
      selectFlatType: "4 ROOM",

      ascending: true,
      chart: "barchart",
    };
  },
  created() {
    // console.log(d3.group(this.data, d => d.town).get("ANG MO KIO")[0]["month"])
  },
  computed: {
    flatTypeData: function () {
      if (this.selectFlatType == "null") {
        return [];
      }

      return this.data.filter((item) => item.flat_type == this.selectFlatType);
    },
    flatTypes: function () {
      const flatTypes = new Set();

      this.data.forEach((element) => {
        flatTypes.add(element["flat_type"]);
      });

      return Array.from(flatTypes);
    },
    towns: function () {
      const flatTypes = new Set();

      this.data.forEach((element) => {
        flatTypes.add(element["town"]);
      });

      return Array.from(flatTypes);
    },
  },
  methods: {
    onSelectChange() {
      this.updateChart();
    },
    setChart(val) {
      this.chart = val;
    },
    setOrder(val) {
      this.ascending = val;
      this.updateChart();
    },
    updateChart() {
      if (this.chart == "boxplot") {
        setTimeout(this.$refs.boxplot.updateBoxPlot(), 3000);
      } else {
        setTimeout(this.$refs.barchart.updateBarChart(), 3000);
      }
    },
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
  margin-top: 60px;
}
</style>
