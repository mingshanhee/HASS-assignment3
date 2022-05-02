<template>
  <!-- Create a div where the graph will take place -->
  <div id="barplot"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "box-plot",
  props: {
    data: Object,
    towns: Object,
    ascending: Boolean,
  },
  data() {
    return {
      top: 10,
      right: 30,
      bottom: 140,
      left: 80,
      width: 1200 - 100,
      height: 800 - 100,
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
      const dataset = [];
      for (let index = 0; index < this.data.length; index++) {
        const element = this.data[index];

        const datapoint = {
          month: element["month"],
          town: element["town"],
          pricePerSqm: element["resale_price"] / element["floor_area_sqm"],
          storey: element["storey"],
        };

        dataset.push(datapoint);
      }

      return dataset;
    },
    subgroups: function () {
      const dict = {};
      for (let index = 0; index < this.towns.length; index++) {
        const elem = this.towns[index];
        dict[elem] = 0;
      }

      for (let index = 0; index < this.data.length; index++) {
        const d = this.data[index];
        dict[d.town] = dict[d.town] + 1;
      }

      const subgroups = [];
      for (const [key, value] of Object.entries(dict)) {
        const datapoint = {
          town: key,
          value: value,
        };

        subgroups.push(datapoint);
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

      subgroups.sort(sort_func);
      return subgroups;
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
        .text("Price Per Square Meter (SGD)");
      // Compute quartiles, median, inter quantile range min and max --> these info are then used to draw the box.
      var sumstat = d3
        .nest() // nest function allows to group the calculation per level of a factor
        .key(function (d) {
          return d.town;
        })
        .rollup(function (d) {
          const q1 = d3.quantile(
            d
              .map(function (g) {
                return g.pricePerSqm;
              })
              .sort(d3.ascending),
            0.25
          );
          const median = d3.quantile(
            d
              .map(function (g) {
                return g.pricePerSqm;
              })
              .sort(d3.ascending),
            0.5
          );
          const q3 = d3.quantile(
            d
              .map(function (g) {
                return g.pricePerSqm;
              })
              .sort(d3.ascending),
            0.75
          );
          const interQuantileRange = q3 - q1;
          const min = q1 - 0.5 * interQuantileRange;
          const max = q3 + 0.5 * interQuantileRange;
          return {
            q1: q1,
            median: median,
            q3: q3,
            interQuantileRange: interQuantileRange,
            min: min,
            max: max,
          };
        })
        .entries(this.plotData);

      // sumstat = await sumstat.sort((a, b) => {
      //   return b["value"]["median"] - a["value"]["median"];
      // });

      const subgroups = this.subgroups.map((elem) => elem["town"]);
      console.log(subgroups);

      // Show the X scale
      var x = d3
        .scaleBand()
        .range([0, this.width])
        .domain(subgroups)
        .paddingInner(1)
        .paddingOuter(0.5);
      this.svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + this.height + ")")
        .call(d3.axisBottom(x))
        .selectAll("text")
        .style("text-anchor", "end")
        .attr("dx", "-.8em")
        .attr("dy", ".15em")
        .attr("transform", "rotate(-65)");

      // Show the Y scale
      var y = d3.scaleLinear().domain([2000, 13000]).range([this.height, 0]);
      this.svg.append("g").call(d3.axisLeft(y));

      // Show the main vertical line
      this.svg
        .selectAll("vertLines")
        .data(sumstat)
        .enter()
        .append("line")
        .attr("x1", function (d) {
          return x(d.key);
        })
        .attr("x2", function (d) {
          return x(d.key);
        })
        .attr("y1", function (d) {
          return y(d.value.min);
        })
        .attr("y2", function (d) {
          return y(d.value.max);
        })
        .attr("stroke", "black")
        .style("width", 40);

      // rectangle for the main box
      var boxWidth = 30;
      this.svg
        .selectAll("boxes")
        .data(sumstat)
        .enter()
        .append("rect")
        .attr("x", function (d) {
          return x(d.key) - boxWidth / 2;
        })
        .attr("y", function (d) {
          return y(d.value.q3);
        })
        .attr("height", function (d) {
          return y(d.value.q1) - y(d.value.q3);
        })
        .attr("width", boxWidth)
        .attr("stroke", "black")
        .style("fill", "#69b3a2");

      // Show the median
      this.svg
        .selectAll("medianLines")
        .data(sumstat)
        .enter()
        .append("line")
        .attr("x1", function (d) {
          return x(d.key) - boxWidth / 2;
        })
        .attr("x2", function (d) {
          return x(d.key) + boxWidth / 2;
        })
        .attr("y1", function (d) {
          return y(d.value.median);
        })
        .attr("y2", function (d) {
          return y(d.value.median);
        })
        .attr("stroke", "black")
        .style("width", 80);
    },
    updateBoxPlot() {
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
