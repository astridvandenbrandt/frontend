<template>
  <h4>Brush Zoom example</h4>
  <h5>{{ msg }}</h5>
  <div class="chart"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "Brush Zoom Example",
  data() {
    return {
      msg: "Hi from brush",
    };
  },
  methods: {
    updateVis(data) {
      console.log("data", data);

      var x = d3
        .scaleUtc()
        .domain(d3.extent(data, (d) => d.date))
        .range([this.margin.left, this.width - this.margin.right]);

      var y = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.value)])
        .range([this.height - this.margin.bottom, this.margin.top]);

      var area = (x, y) =>
        d3
          .area()
          .defined((d) => !isNaN(d.value))
          .x((d) => x(d.date))
          .y0(y(0))
          .y1((d) => y(d.value));

      var xAxis = (g) =>
        g
          .attr("transform", `translate(0,${this.height - this.margin.bottom})`)
          .call(
            d3
              .axisBottom(x)
              .ticks(this.width / 80)
              .tickSizeOuter(0)
          );

      //   var yAxis = (g) =>
      //     g
      //       .attr("transform", `translate(${this.margin.left},0)`)
      //       .call(d3.axisLeft(y))
      //       .call((g) => g.select(".domain").remove())
      //       .call((g) =>
      //         g
      //           .selectAll(".title")
      //           .data([title])
      //           .join("text")
      //           .attr("class", "title")
      //           .attr("x", -this.margin.left)
      //           .attr("y", 10)
      //           .attr("fill", "currentColor")
      //           .attr("text-anchor", "start")
      //           .text(title)
      //       );

      const defaultSelection = [
        x(d3.utcYear.offset(x.domain()[1], -1)),
        x.range()[1],
      ];

      this.svg.append("g").call(xAxis, x, this.focusHeight);

      this.svg
        .append("path")
        .datum(data)
        .attr("fill", "steelblue")
        .attr(
          "d",
          area(x, y.copy().range([this.focusHeight - this.margin.bottom, 4]))
        );

      const brush = d3
        .brushX()
        .extent([
          [this.margin.left, 0.5],
          [this.width - this.margin.right, this.focusHeight - this.margin.bottom + 0.5],
        ])
        .on("brush", brushed)
        .on("end", brushended);

      const gb = this.svg
        .append("g")
        .call(brush)
        .call(brush.move, defaultSelection);

      function brushed({ selection }) {
        if (selection) {
          this.svg.property(
            "value",
            selection.map(x.invert, x).map(d3.utcDay.round)
          );
          this.svg.dispatch("input");
        }
      }

      function brushended({ selection }) {
        if (!selection) {
          gb.call(brush.move, defaultSelection);
        }
      }
    },
  },
  mounted() {
    var margin = { top: 20, right: 20, bottom: 30, left: 40 },
      focusHeight = 100,
      height = 450 - margin.top - margin.bottom,
      constWidth = d3.select(".chart").node().clientWidth,
      width = constWidth - margin.left - margin.right;

    this.margin = margin;
    this.width = width;
    this.height = height;
    this.focusHeight = focusHeight;

    //Appends the svg to the chart-container div
    var svg = d3
      .select(".chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    this.svg = svg;
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style></style>
