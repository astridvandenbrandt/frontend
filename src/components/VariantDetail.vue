<template>
  <!-- <h4>Sequence Variants</h4> -->
  <!-- <h5>{{ msg }}</h5> -->
  <!-- Initialize a select button -->
  <select id="selectButtonMSA"> <option  disabled selected>sort by: </option></select>
  <div id="my_dataviz"></div>
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

      var vis = this.svg;
      var visX = this.xScale;
      var visY = this.yScale;
      

      // Labels of row and columns -> unique identifier of the column called 'group' and 'variable'
      var myGroups = d3
        .map(data, function(d) {
          return d.pos;
        })
        .filter(unique);
      console.log(myGroups);

      // add domains and build axis
      visX
        .domain(myGroups);

      var xAxis = d3
        .axisTop(visX)
        .tickSize(10)
        .tickValues(
          visX.domain().filter(function(d, i) {
            return !(i % 5);
          })
        ); //show every fifth position

      // default sorting rows
      visY.domain(
        d3
          .map(data, function(d) {
            return d.accession;
          })
          .filter(unique)
          .sort(d3.descending),
      ); 

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

      
      // call vis axes 
      vis
        .append("g")
        .style("font-size", 15)
        // .attr("transform", "translate(0," + this.height + ")")
        .call(xAxis)
        .select(".domain")
        .remove();

        vis
        .append("g")
        .style("font-size", 15)
        .call(d3.axisLeft(visY).tickSize(0))
        .select(".domain")
        .remove();

  
      // add the squares
      vis
        .selectAll()
        .data(data, function(d) {
          // console.log('d.group + ":" + d.variable', d.group + ":" + d.variable)
          return d.pos + ":" + d.accesion;
        })
        .enter()
        .append("rect")
        .attr("x", function(d) {
          // console.log("xScale(d.pos)", xScale(d.pos), d.pos);
          return visX(d.pos);
        })
        .attr("y", function(d) {
          // console.log("yScale(d.base)", yScale(d.accession), d.accession);
          return visY(d.accession);
        })
        .attr("rx", 4)
        .attr("ry", 4)
        .attr("width", visX.bandwidth())
        .attr("height", visY.bandwidth())
        .style("fill", function(d) {
          return colors[d.base];
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8)
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);



      // Three functions that change the tooltip when user hover / move / leave a cell
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


    // Precompute the orders.
    var orders = {
      alpha_asc: "alphabetical",
      alpha_desc: "reversed alphabetical",
      phylo: "phylogeny",

    };

    this.orders = orders;

    // add the options to the button
    d3.select("#selectButtonMSA")
      .selectAll("myOptions")
      .data(Object.keys(orders))
      .enter()
      .append("option")
      .text(function(d) {
        return orders[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value returned by the button

    // var orders = {
    //     default: [
    //       "8_Tsu-0",
    //       "8_Altai-5",
    //       "8_Sha",
    //       "7_Ler",
    //       "6_Kyo",
    //       "5_Sku-30",
    //       "5_Ler",
    //       "5_Gro-3",
    //       "5_Eri-1",
    //       "4_Cvi-0",
    //       "3_C24",
    //       "2_An-1",
    //       "1_Kas-1",
    //       "1_Col-0",
    //     ],
    //     alphabetical: d3
    //       .map(data, function(d) {
    //         return d.accession;
    //       })
    //       .filter(unique)
    //       .sort(d3.descending),
    //     phylogeny: [
    //       "6_Kyo",
    //       "5_Sku-30",
    //       "5_Ler",
    //       "5_Gro-3",
    //       "5_Eri-1",
    //       "7_Ler",
    //       "8_Sha",
    //       "8_Tsu-0",
    //       "8_Altai-5",
    //       "3_C24",
    //       "4_Cvi-0",
    //       "1_Col-0",
    //       "1_Kas-1",
    //       "2_An-1",
    //     ],
    //   };

      console.log("orders", Object.keys(orders));



    // Build scales:
    var xScale = d3
        .scaleBand()
        .range([0, width])
        .padding(0.05);

    this.xScale = xScale;

    var yScale = d3
        .scaleBand()
        .range([height, 0])
        .padding(0.05);

    this.yScale = yScale;

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
#selectButtonMSA {
  position: absolute;
  left: 0;
  margin-left: 30px;
  margin-top: 10px;
}
</style>
