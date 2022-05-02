<template>
  <!-- Create a div where the graph will take place -->
  <div id="barplot"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "bar-chart",
  props: {
    data: Object,
    towns: Object,
    ascending: Boolean,
  },
  data() {
    return {
      top: 20,
      right: 30,
      bottom: 140,
      left: 70,
      width: 1200 - 100,
      height: 480 - 30,
    };
  },
  async mounted() {
    this.svg = d3
      .select("#barplot")
      .append("svg")
      .attr("width", this.width + this.left + this.right)
      .attr("height", this.height + this.top + this.bottom)
      .append("g")
      .attr("transform", `translate(${this.left},${this.top})`);

    await this.setupBoxPlot();
  },
  computed: {
    plotData: function () {
      const dict = {};
      for (let index = 0; index < this.towns.length; index++) {
        const elem = this.towns[index];
        dict[elem] = 0;
      }

      for (let index = 0; index < this.data.length; index++) {
        const d = this.data[index];
        dict[d.town] = dict[d.town] + 1;
      }

      const plotData = [];
      for (const [key, value] of Object.entries(dict)) {
        const datapoint = {
          town: key,
          value: value,
        };

        plotData.push(datapoint);
      }

      var sort_func = null;
      if (this.ascending) {
        sort_func = function (b, a) {
          return a.value - b.value;
        };
      } else {
        sort_func = function (b, a) {
          return b.value - a.value;
        };
      }

      plotData.sort(sort_func);

      return plotData;
    },
    maxY: function () {
      var max = 0;
      for (let index = 0; index < this.plotData.length; index++) {
        const elem = this.plotData[index];
        for (const [key, value] of Object.entries(elem)) {
          if (key == "town") continue;

          if (value > max) max = value;
        }
      }
      return max;
    },
  },
  methods: {
    capitalize(str) {
      return str[0].toUpperCase() + str.slice(1).toLowerCase();
    },
    async setupBoxPlot() {
      // x-axis label
      this.svg
        .append("text")
        .attr("class", "x label")
        .attr("text-anchor", "end")
        .attr("x", this.width / 1.75)
        .attr("y", this.height + this.bottom - this.top)
        .text("Town Areas");

      // y-axis label
      this.svg
        .append("text")
        .attr("class", "y label")
        .attr("text-anchor", "end")
        .attr("x", -this.height / 3.5)
        .attr("y", -50)
        .attr("transform", "rotate(-90)")
        .text("Number of Transaction Records");
      // X axis
      var x = d3
        .scaleBand()
        .range([0, this.width])
        .domain(
          this.plotData.map(function (d) {
            return d.town;
          })
        )
        .padding(0.2);
      this.svg
        .append("g")
        .attr("transform", "translate(0," + this.height + ")")
        .call(d3.axisBottom(x))
        .selectAll("text")
        .attr("transform", "translate(-10,0)rotate(-45)")
        .style("text-anchor", "end");

      // Add Y axis
      var y = d3.scaleLinear().domain([0, this.maxY]).range([this.height, 0]);
      this.svg.append("g").call(d3.axisLeft(y));

      // Bars
      this.svg
        .selectAll("mybar")
        .data(this.plotData)
        .enter()
        .append("rect")
        .attr("x", function (d) {
          return x(d.town);
        })
        .attr("y", function (d) {
          return y(d.value);
        })
        .attr("width", x.bandwidth())
        .attr("height", (d) => {
          return this.height - y(d.value);
        })
        .attr("fill", "#69b3a2");
    },
    updateBarChart() {
      this.svg.selectAll("*").remove();
      this.setupBoxPlot();
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
