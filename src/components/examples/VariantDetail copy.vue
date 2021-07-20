<template>
  <!-- <h4>Sequence Variants</h4> -->
  <!-- <h5>{{ msg }}</h5> -->
  <div id="my_dataviz"></div>
  <!-- <div id="dataset-picker"></div> -->
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantDetail",
  data() {
    return {
      msg: "Hi from sequene variants component",
    };
  },
  methods: {
    updateVis(data) {

      const unique = (value, index, self) => {
        return self.indexOf(value) === index;
      };

      // Labels of row and columns -> unique identifier of the column called 'group' and 'variable'
      var myGroups = d3
        .map(data, function(d) {
          return d.group;
        })
        .filter(unique);
      // console.log(myGroups);

      var myVars = d3
        .map(data, function(d) {
          return d.variable;
        })
        .filter(unique);
      // console.log(myVars);

      // Build X scales and axis:
      var xScale = d3
        .scaleBand()
        .range([0, this.width])
        .domain(myGroups)
        .padding(0.05);
      
      this.svg
        .append("g")
        .style("font-size", 15)
        // .attr("transform", "translate(0," + this.height + ")")
        .call(d3.axisTop(xScale).tickSize(0))
        .select(".domain")
        .remove();

      // Build Y scales and axis:
      var yScale = d3
        .scaleBand()
        .range([this.height, 0])
        .domain(myVars)
        .padding(0.05);
      
      this.svg
        .append("g")
        .style("font-size", 15)
        .call(d3.axisLeft(yScale).tickSize(0))
        .select(".domain")
        .remove();

      // Build color scale
      var myColor = d3
        .scaleSequential()
        .interpolator(d3.interpolateInferno)
        .domain([1, 100]);

      // create a tooltip
      var tooltip = d3
        .select("#my_dataviz")
        .append("div")
        .style("opacity", 0)
        .attr("class", "tooltip")
        .style("background-color", "white")
        .style("border", "solid")
        .style("border-width", "2px")
        .style("border-radius", "5px")
        .style("padding", "5px");

      // Three function that change the tooltip when user hover / move / leave a cell
      var mouseover = function() {
        tooltip.style("opacity", 1);
        d3.select(this)
          .style("stroke", "black")
          .style("opacity", 1)
      };

      var mousemove = function(event, d) {
        tooltip
          .html("The exact value of<br>this cell is: " + d.value)
          .style("left", d3.pointer(event)[0] + 70 + "px")
          .style("top", d3.pointer(event)[1] + 100 + "px");
      };
      var mouseleave = function() {
        tooltip.style("opacity", 0);
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      // add the squares
      this.svg
        .selectAll()
        .data(data, function(d) {
          console.log('d.group + ":" + d.variable', d.group + ":" + d.variable)
          return d.group + ":" + d.variable;
        })
        .enter()
        .append("rect")
        .attr("x", function(d) {
          console.log('xScale(d.group)', xScale(d.group), d.group)
          return xScale(d.group);
        })
        .attr("y", function(d) {
          console.log('yScale(d.variable)', yScale(d.variable), d.group)
          return yScale(d.variable);
        })
        .attr("rx", 4)
        .attr("ry", 4)
        .attr("width", xScale.bandwidth())
        .attr("height", yScale.bandwidth())
        .style("fill", function(d) {
          return myColor(d.value);
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8)
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);
    },
  },
  mounted() {
    // set the dimensions and margins of the graph
    var margin = { top: 80, right: 25, bottom: 30, left: 40 },
      width = 400 - margin.left - margin.right,
      height = 450 - margin.top - margin.bottom;

    this.width = width;
    this.height = height;

    // append the svg object to the body of the page
    var svg = d3
      .select("#my_dataviz")
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
<style>
</style>
