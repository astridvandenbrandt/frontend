<template>
  <!-- <h4>Sequence Variants</h4> -->
  <!-- <h5>{{ msg }}</h5> -->
  <!-- Initialize a select button -->
  <select id="selectButton"></select>
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

      // Precompute the orders.
      var orders = {
        default: [
          "8_Tsu-0",
          "8_Altai-5",
          "8_Sha",
          "7_Ler",
          "6_Kyo",
          "5_Sku-30",
          "5_Ler",
          "5_Gro-3",
          "5_Eri-1",
          "4_Cvi-0",
          "3_C24",
          "2_An-1",
          "1_Kas-1",
          "1_Col-0",
        ],
        alphabetical: d3
          .map(data, function(d) {
            return d.accession;
          })
          .filter(unique)
          .sort(d3.descending),
      };

      console.log("orders", Object.keys(orders));

      // add the options to the button
      d3.select("#selectButton")
        .selectAll("myOptions")
        .data(Object.keys(orders))
        .enter()
        .append("option")
        .text(function(d) {
          console.log("d button", d);
          return d;
        }) // text showed in the menu
        .attr("value", function(d) {
          console.log("button value", d);
          return orders[d];
        }); // corresponding value returned by the button

      // Labels of row and columns -> unique identifier of the column called 'group' and 'variable'
      var myGroups = d3
        .map(data, function(d) {
          return d.pos;
        })
        .filter(unique);
      console.log(myGroups);

      // Build X scales and axis:
      var xScale = d3
        .scaleBand()
        .range([0, this.width])
        .domain(myGroups)
        .padding(0.05);

      var xAxis = d3
        .axisTop(xScale)
        .tickSize(10)
        .tickValues(
          xScale.domain().filter(function(d, i) {
            return !(i % 5);
          })
        ); //show every fifth position

      var chart = this.svg

      chart
        .append("g")
        .style("font-size", 15)
        // .attr("transform", "translate(0," + this.height + ")")
        .call(xAxis)
        .select(".domain")
        .remove();

      // Build Y scales and axis:
      var yScale = d3
        .scaleBand()
        .range([this.height, 0])
        .padding(0.05);

      yScale.domain(orders.default); //default

      chart
        .append("g")
        .style("font-size", 15)
        .call(d3.axisLeft(yScale).tickSize(0))
        .select(".domain")
        .remove();

      // // Build color scale
      // var myColor = d3
      //   .scaleSequential()
      //   .interpolator(d3.interpolateInferno)
      //   .domain([1, 100]);

      // build color scale categories
      var colors = {
        A: "#e78ac3",
        a: "#e78ac3",
        G: "#fc8d62",
        g: "#fc8d62",
        C: "#8da0cb",
        c: "#8da0cb",
        T: "#66c2a5",
        t: "#66c2a5",
        "-": "lightgray",
      };

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
          .style("opacity", 1);
      };

      var mousemove = function(event, d) {
        tooltip
          .html("base: "+d.base)
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
      chart
        .selectAll()
        .data(data, function(d) {
          // console.log('d.group + ":" + d.variable', d.group + ":" + d.variable)
          return d.pos + ":" + d.accesion;
        })
        .enter()
        .append("rect")
        .attr("x", function(d) {
          console.log("xScale(d.pos)", xScale(d.pos), d.pos);
          return xScale(d.pos);
        })
        .attr("y", function(d) {
          console.log("yScale(d.base)", yScale(d.accession), d.accession);
          return yScale(d.accession);
        })
        .attr("rx", 4)
        .attr("ry", 4)
        .attr("width", xScale.bandwidth())
        .attr("height", yScale.bandwidth())
        .style("fill", function(d) {
          return colors[d.base];
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8)
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);

      // A function that update the chart
      function update(selectedGroup) {
        console.log("update!", selectedGroup);
        // Give these new data to update scale
        yScale.domain(selectedGroup);
        chart.call(d3.axisLeft(yScale).tickSize(0));
        
      }

      // When the button is changed, run the updateChart function
      d3.select("#selectButton").on("change", function() {
        var selectedOption = d3.select(this).property("value");
        console.log("selectedOption", selectedOption);
        // run the updateChart function with this selected option
        update(selectedOption);
      });
    },
  },
  mounted() {
    // set the dimensions and margins of the graph
    var margin = { top: 80, right: 20, bottom: 30, left: 80 },
      width =
        d3.select("#my_dataviz").node().clientWidth -
        margin.left -
        margin.right,
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
#selectButton {
  position: absolute;
  left: 0;
  margin-left: 30px;
  margin-top: 10px;
}
</style>
