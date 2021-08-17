<template>
  <h4>Sequence Variants</h4>
  <!-- <h5>{{ msg }}</h5> -->
  <div class="gene_chart"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantOverview",
  data() {
    return {
      msg: "Hi from sequene variants overview",
      // range: [x0, x1]
    };
  },
  methods: {
    updateVis(data) {
      console.log("data overview", data);
      // var max = d3.max(data, function(d) {
      //   return d.percent;
      // });
      // console.log("maxvalue", max);

      // var total = d3.rollup(
      //   data,
      //   (v) => d3.sum(v, (d) => d.value),
      //   (d) => d.group
      // );
      // var total_array = Array.from(total);
      // console.log("total", total);
      // console.log("total", total_array);
      // var max_total = d3.max(total_array, function(d) {
      //   return d[1];
      // });
      // console.log("max total", max_total);

      // data.forEach(function(d) {
      //   d.value = +d.value;
      // });

      // //Organizes the data
      // var maxX = d3.max(data, function(d) {
      //   return d.value;
      // });
      // console.log("maxvalue", maxX);

      //Creates the xScale
      var xScale = d3.scaleLinear().range([0, this.width]);

      //Creates the yScale
      var yScale = d3.scaleLinear().range([this.height, 0]);

      //Defines the xScale max
      xScale.domain([0, 4383]);

      //Defines the yScale max
      yScale.domain([0, 50]);

      //Defines the y axis styles`
      var xAxis = d3
        .axisTop()
        .scale(xScale)
        .tickPadding(8)
        .tickValues([0, 500, 1000, 1500, 2000, 2500, 3000, 3500, 4000, 4383]);

      //Appends the x axis
      this.svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + 50 + ")")
        .call(xAxis);

      //Binds data to strips
      this.svg
        .selectAll("line.percent")
        .data(data)
        .enter()
        .append("line")
        .attr("class", "percentline")
        .attr("x1", function(d) {
          // console.log("x1", d.position, xScale(d.position));
          return xScale(d.position);
        })
        .attr("x2", function(d) {
          return xScale(d.position);
        })
        .attr("y1", 50)
        .attr("y2", 100);

      this.svg
        .append("g")
        .attr("class", "brush")
        .call(this.brush)
        .call(this.brush.move, [3700, 3900].map(xScale));

      this.brush.on("start brush end", brushed);

      function brushed({ selection }) {
        console.log("selection", { selection });
          const rangeSelected = selection.map(xScale.invert);
          console.log("[x0, x1]", rangeSelected, Math.round(rangeSelected[0]), Math.round(rangeSelected[1])) ;
    
      }

      // removes handle to resize the brush
      d3.selectAll(".brush>.handle").remove();
      // removes crosshair cursor
      d3.selectAll(".brush>.overlay").remove();
    },
  },
  mounted() {
    //Margin conventions
    var margin = { top: 0, right: 20, bottom: 25, left: 20 };

    var constWidth = d3.select(".gene_chart").node().clientWidth;

    var width = constWidth - margin.left - margin.right,
      height = 125 - margin.top - margin.bottom;

    console.log("width", width);
    console.log("height", height);

    var focusHeight = 50;
    console.log("focusHeight", focusHeight);

    this.width = width;
    this.height = height;

    const brush = d3.brushX().extent([
      [0, focusHeight],
      [width, height],
    ]);

    this.brush = brush;

    //Appends the svg to the chart-container div
    var svg = d3
      .select(".gene_chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    this.svg = svg;

    //Appends background color
    svg
      .append("rect")
      .attr("class", "background-gene")
      .attr("width", width)
      .attr("height", 50)
      .attr("transform", "translate(0," + 50 + ")");
    // .attr("fill", "blue")
    // .attr("opacity", 0.1);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.background-gene {
  fill: lightsteelblue;
  opacity: 0.1;
}

.percentline {
  stroke: black;
  stroke-width: 1;
  opacity: 0.4;
}

.selection {
    stroke: white;
    fill: blue;
    fill-opacity: .165;
}
</style>
