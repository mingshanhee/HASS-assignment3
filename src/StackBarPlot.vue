<template>
  <!-- Create a div where the graph will take place -->
  <div id="barplot"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "stack-bar-plot",
  props: {
    data: Object,
  },
  data() {
    return {
      top: 10,
      right: 30,
      bottom: 20,
      left: 50,
      width: 860 - 80,
      height: 800 - 30,
    };
  },
  mounted() {
    // d3.csv(
    //   "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/data_stacked.csv"
    // ).then(function (data) {
    //   // List of subgroups = header of the csv files = soil condition here
    //   console.log(data);
    //   const subgroups = data.columns.slice(1);
    //   console.log(subgroups);
    //   // List of groups = species here = value of the first column called group -> I show them on the X axis
    //   const groups = data.map((d) => d.group);
    //   console.log(groups);
    // });
    this.svg = d3
      .select("#barplot")
      .append("svg")
      .attr("width", this.width + this.left + this.right)
      .attr("height", this.height + this.top + this.bottom)
      .append("g")
      .attr("transform", `translate(${this.left},${this.top})`);

    this.setupStackBarChart();
    this.updateStackBarChart();
  },
  computed: {
    regionData: function () {
      if (this.selectRegion == "null") {
        return this.data;
      }

      return this.data.filter((item) => item.town == this.selectRegion);
    },
    regions: function () {
      const regions = new Set();

      this.data.forEach((element) => {
        regions.add(element["town"]);
      });

      return regions;
    },
    plotData: function () {
      const counter = {};
      for (let index = 0; index < this.data.length; index++) {
        const element = this.data[index];
        const month = element["month"];
        const flatType = element["flat_type"];

        if (!counter[month]) {
          counter[month] = {
            "1 ROOM": 0,
            "2 ROOM": 0,
            "3 ROOM": 0,
            "4 ROOM": 0,
            "5 ROOM": 0,
            EXECUTIVE: 0,
          };
        }

        counter[month][flatType] += 1;
      }

      const groups = [];
      for (const [key, value] of Object.entries(counter)) {
        const group = {
          group: key,
        };

        for (let index = 0; index < this.subgroups.length; index++) {
          const element = this.subgroups[index];
          group[element] = value[element];
        }

        groups.push(group);
      }

      return groups;
    },
    groups: function () {
      // List of groups = species here = value of the first column called group -> I show them on the X axis
      const groups = this.plotData.map((d) => d.group);
      groups.sort();
      return groups;
    },
    subgroups: function () {
      return ["1 ROOM", "2 ROOM", "3 ROOM", "4 ROOM", "5 ROOM", "EXECUTIVE"];
    },
    maxY: function() {
      var max = 0;
      for (let index = 0; index < this.plotData.length; index++) {
        const elem = this.plotData[index];
        for (const [key, value] of Object.entries(elem)) {
          if (key == "group")
            continue

          if (value > max)
            max = value
        }
      }
      return max
    }
  },
  methods: {
    capitalize(str) {
      return str[0].toUpperCase() + str.slice(1).toLowerCase();
    },
    test() {
      const counter = {};
      for (let index = 0; index < this.data.length; index++) {
        const element = this.data[index];
        const month = element["month"];
        const flatType = element["flat_type"];

        if (!counter[month]) {
          counter[month] = {
            "1 ROOM": 0,
            "2 ROOM": 0,
            "3 ROOM": 0,
            "4 ROOM": 0,
            "5 ROOM": 0,
            EXECUTIVE: 0,
          };
        }

        counter[month][flatType] += 1;
      }

      const groups = [];
      for (const [key, value] of Object.entries(counter)) {
        const group = {
          group: key,
        };

        for (let index = 0; index < this.subgroups.length; index++) {
          const element = this.subgroups[index];
          group[element] = value[element];
        }

        groups.push(group);
      }

      return groups;
    },
    setupStackBarChart() {
      // Add X axis
      const x = d3
        .scaleBand()
        .domain(this.groups)
        .range([0, this.width])
        .padding([0.2]);

      this.svg
        .append("g")
        .attr("transform", `translate(0, ${this.height})`)
        .call(d3.axisBottom(x).tickSize(0));

      // Add Y axis
      
      const y = d3.scaleLinear().domain([0, this.maxY]).range([this.height, 0]);

      this.svg.append("g").call(d3.axisLeft(y));

      // Another scale for subgroup position?
      const xSubgroup = d3
        .scaleBand()
        .domain(this.subgroups)
        .range([0, x.bandwidth()])
        .padding([0.05]);

      // color palette = one color per subgroup
      const color = d3
        .scaleOrdinal()
        .domain(this.subgroups)
        .range([
          "#e41a1c",
          "#377eb8",
          "#4daf4a",
          "#e41a1c",
          "#377eb8",
          "#4daf4a",
        ]);
      this.svg
        .append("g")
        .selectAll("g")
        // Enter in data = loop group per group
        .data(this.plotData)
        .join("g")
        .attr("transform", (d) => `translate(${x(d.group)}, 0)`)
        .selectAll("rect")
        .data(function (d) {
          return [
            "1 ROOM",
            "2 ROOM",
            "3 ROOM",
            "4 ROOM",
            "5 ROOM",
            "EXECUTIVE",
          ].map(function (key) {
            return { key: key, value: d[key] };
          });
        })
        .join("rect")
        .attr("x", (d) => xSubgroup(d.key))
        .attr("y", (d) => y(d.value))
        .attr("width", xSubgroup.bandwidth())
        .attr("height", (d) => this.height - y(d.value))
        .attr("fill", (d) => color(d.key));
    },
    updateStackBarChart() {
      this.svg.selectAll("*").remove();
      this.setupStackBarChart();
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
