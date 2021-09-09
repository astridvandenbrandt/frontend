<template>
  <h4>Testing Brush</h4>
  <!-- <h5>{{ msg }}</h5> -->
  <div class="chart_brush_example"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "Testing Brush",
  data() {
    return {
      msg: "Hi from brush",
    };
  },
  methods: {
    updateVis(data) {
      console.log("data brush test", data);

      data.forEach(function(d) {
        d.value = +d.value;
      });
    },
  },
  mounted() {
    var data = [];

    let num = d3.randomNormal(1, 50);
    for (let i = 0; i < 20; i++) {
      data.push({
          id: i +1,
          y: num()});
    }

    // var data = randomData(200);
    console.log("data brush fixed", data);

    //Margin conventions
    var margin = { top: 10, right: 20, bottom: 20, left: 20 };

    var constWidth = d3.select(".chart_brush_example").node().clientWidth;
    var width = constWidth - margin.left - margin.right;

    var height = 300;

    // var margin = {top: 20, right: 20, bottom: 30, left: 50},
    //     width = 560 - margin.left - margin.right,
    //     height = 260 - margin.top - margin.bottom;

    var x = d3.scaleLinear().range([0, width]);

    var y = d3.scaleLinear().range([height, 0]);

    var line = d3
      .line()
      .x(function(d) {
        return x(d.id);
      })
      .y(function(d) {
        return y(d.y);
      });

    var brush = d3.brushX().extent([
      [0, 0],
      [width, height],
    ]);

    //Appends the svg to the chart-container div
    var svg = d3
      .select(".chart_brush_example")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    data.forEach(function(d) {
      Object.keys(data[0]).forEach(function(k) {
        d[k] = +d[k];
      });
    });

    x.domain(
      d3.extent(data, function(d) {
        return d.id;
      })
    );
    y.domain(
      d3.extent(data, function(d) {
        return d.y;
      })
    );

    svg
      .append("g")
      .attr("class", "axis axis--x")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x));

    svg
      .append("g")
      .attr("class", "axis axis--y")
      .call(d3.axisLeft(y));

    svg
      .append("path")
      .datum(data)
      .attr("class", "line")
      .attr("d", line);

    svg
      .append("g")
      .attr("class", "brush")
      .call(brush)
      .call(brush.move, [x(60), x(120)]);

    // removes handle to resize the brush
    d3.selectAll(".brush>.handle").remove();
    // removes crosshair cursor
    d3.selectAll(".brush>.overlay").remove();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style></style>
