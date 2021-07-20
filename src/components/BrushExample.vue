<template>
  <h4>Brush example</h4>
  <!-- <h5>{{ msg }}</h5> -->
  <div class="chart_brush"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "Brush Example",
  data() {
    return {
      msg: "Hi from brush",
    };
  },
  methods: {
    updateVis(data) {
      console.log("data overview", data);

      data.forEach(function(d) {
        d.value = +d.value;
      });
    },
  },
  mounted() {
    //Margin conventions
    var margin = { top: 10, right: 20, bottom: 20, left: 20 };

    var constWidth = d3.select(".chart_brush").node().clientWidth;
    var width = constWidth - margin.left - margin.right;

    var height = 150 - margin.top - margin.bottom;

    this.height = height;

    var ry = d3.randomNormal(height / 2, height / 12);

    var x = d3.scaleLinear([0, 10], [margin.left, width - margin.right]);
    var rx = d3.randomUniform(...x.domain());

    console.log(ry, rx);

    var xAxis = (g) =>
      g
        .attr("transform", `translate(0,${height - margin.bottom})`)
        .call(d3.axisBottom(x));

    //Appends the svg to the chart-container div
    var svg = d3
      .select(".chart_brush")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    const brush = d3
      .brushX()
      .extent([
        [margin.left, margin.top],
        [width - margin.right, height - margin.bottom],
      ])
      .on("start brush end", brushed);

    var circ = svg
      .append("g")
      .attr("fill-opacity", 0.2)
      .selectAll("circle")
      .data(Float64Array.from({ length: 800 }, rx))
      .join("circle")
      .attr("transform", (d) => `translate(${x(d)},${ry()})`)
      .attr("r", 3.5);

    svg.append("g").call(xAxis);

    svg
      .append("g")
      .call(brush)
      .call(brush.move, [3, 5].map(x));
      // .on("dblclick", dblclicked);

    // function dblclicked() {
    //   const selection = d3.brushSelection(this) ? null : x.range();
    //   d3.select(this).call(brush.move, selection);
    // }

    function brushed({ selection }) {
      if (selection === null) {
        circ.attr("stroke", null);
      } else {
        const [x0, x1] = selection.map(x.invert);
        circ.attr("stroke", (d) => (x0 <= d && d <= x1 ? "blue" : null));
      }
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style> 
</style>
