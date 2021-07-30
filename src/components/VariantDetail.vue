<template>
  <div id="selectMSA">
    <label for="selectButtonMSA"> Sort By: </label>
    <select id="selectButtonMSA"></select>
  </div>
  <div id="msa_chart"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantDetail",
  // data() {
  // },
  methods: {
    updateVis(data) {
      const unique = (value, index, self) => {
        return self.indexOf(value) === index;
      };

      var vis = this.svg;
      var visX = this.xScale;
      var visY = this.yScale;

      // Create labels for gene positions
      var genePositions = d3
        .map(data, function(d) {
          return d.pos;
        })
        .filter(unique);
      console.log(genePositions);

      // Add domains and build axis
      visX.domain(genePositions);

      var xAxis = d3
        .axisTop(visX)
        .tickSize(10)
        .tickValues(
          visX.domain().filter(function(d, i) {
            return !(i % 5);
          })
        ); //show every fifth position

      // Default sorting rows
      visY.domain(
        d3
          .map(data, function(d) {
            return d.accession;
          })
          .filter(unique)
          .sort(d3.descending)
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
        "*": "grey"
      };

      // create a tooltip
      var tooltip = d3
        .select("#msa_chart")
        .append("div")
        .style("opacity", 0)
        .attr("class", "tooltip")
        .style("background-color", "white")
        .style("border", "solid")
        .style("border-width", "2px")
        .style("border-radius", "5px")
        .style("padding", "5px");

      // Call vis axes
      vis
        .append("g")
        .attr("class", "x-axis")
        .style("font-size", 15)
        // .attr("transform", "translate(0," + this.height + ")")
        .call(xAxis)
        .select(".domain")
        .remove();

      vis
        .append("g")
        .attr("class", "y-axis")
        .style("font-size", 15)
        .call(d3.axisLeft(visY).tickSize(0))
        .select(".domain")
        .remove();

      // Add the squares
      vis
        .selectAll()
        .data(data, function(d) {
          return d.pos + ":" + d.accesion;
        })
        .enter()
        .append("rect")
        .attr("class", "cell")
        .attr("x", function(d) {
          return visX(d.pos);
        })
        .attr("y", function(d) {
          return visY(d.accession);
        })
        .attr("rx", 2)
        .attr("ry", 2)
        .attr("width", visX.bandwidth())
        .attr("height", visY.bandwidth())
        .style("fill", function(d) {
          return colors[d.base];
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8);

      vis
        .selectAll(".cell")
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);

      // Three functions that change the tooltip when user hover / move / leave a cell
      var mouseover = function() {
        tooltip.style("opacity", 1);
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "2px")
          .style("opacity", 0.8);
      };

      var mousemove = function(event, d) {
        tooltip
          .html("base: " + d.base)
          .style("left", d3.pointer(event)[0] + 65 + "px")
          .style("top", d3.pointer(event)[1] + 150 + "px");
      };
      var mouseleave = function() {
        tooltip.style("opacity", 0);
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      // Change row ordering based on select
      d3.select("#selectButtonMSA").on("change", function() {
        var selected = d3.select("#selectButtonMSA").node().value;
        console.log("selected order MSA: ", selected);

        if (selected === "alpha_asc") {
          // sorting rows
          visY.domain(
            d3
              .map(data, function(d) {
                return d.accession;
              })
              .filter(unique)
              .sort(d3.descending)
          );
          vis
            .selectAll(".y-axis")
            .transition()
            .duration(800)
            .call(d3.axisLeft(visY).tickSize(0))
            .on("start", function() {
              vis.select(".y-axis .domain").remove();
            });
          vis
            .selectAll(".cell")
            .transition()
            .duration(800)
            .attr("x", function(d) {
              return visX(d.pos);
            })
            .attr("y", function(d) {
              return visY(d.accession);
            });
          vis
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "alpha_desc") {
          // sorting rows
          visY.domain(
            d3
              .map(data, function(d) {
                return d.accession;
              })
              .filter(unique)
              .sort(d3.ascending)
          );
          vis
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visY).tickSize(0))
            .on("start", function() {
              vis.select(".y-axis .domain").remove();
            });
          vis
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visX(d.pos);
            })
            .attr("y", function(d) {
              return visY(d.accession);
            });
          vis
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "phylo") {
          // sorting rows
          visY.domain([
            "2_An-1",
            "1_Kas-1",
            "1_Col-0",
            "4_Cvi-0",
            "3_C24",
            "8_Tsu-0",
            "8_Altai-5",
            "8_Sha",
            "7_Ler",
            "5_Sku-30",
            "5_Ler",
            "5_Gro-3",
            "5_Eri-1",
            "6_Kyo",
          ]);
          vis
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visY).tickSize(0))
            .on("start", function() {
              vis.select(".y-axis .domain").remove();
            });
          vis
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visX(d.pos);
            })
            .attr("y", function(d) {
              return visY(d.accession);
            });
          vis
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "phylo_rev") {
          // sorting rows
          visY.domain([
            "6_Kyo",
            "5_Sku-30",
            "5_Ler",
            "5_Gro-3",
            "5_Eri-1",
            "7_Ler",
            "8_Sha",
            "8_Tsu-0",
            "8_Altai-5",
            "3_C24",
            "4_Cvi-0",
            "1_Col-0",
            "1_Kas-1",
            "2_An-1",
          ]);
          vis
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visY).tickSize(0))
            .on("start", function() {
              vis.select(".y-axis .domain").remove();
            });
          vis
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visX(d.pos);
            })
            .attr("y", function(d) {
              return visY(d.accession);
            });
          vis
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }

        if (selected === "ref_first") {
          // sorting rows
          visY.domain([
            "8_Tsu-0",
            "8_Altai-5",
            "5_Gro-3",
            "5_Ler",
            "5_Sku-30",
            "1_Kas-1",
            "2_An-1",
            "8_Sha",
            "5_Eri-1",
            "6_Kyo",
            "3_C24",
            "7_Ler",
            "4_Cvi-0",
            "1_Col-0",
          ]);
          vis
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visY).tickSize(0))
            .on("start", function() {
              vis.select(".y-axis .domain").remove();
            });
          vis
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visX(d.pos);
            })
            .attr("y", function(d) {
              return visY(d.accession);
            });
          vis
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
      });
    },
  },
  mounted() {
    // set the dimensions and margins of the graph
    var margin = { top: 80, right: 20, bottom: 30, left: 80 },
      width =
        d3.select("#msa_chart").node().clientWidth -
        margin.left -
        margin.right,
      height = 450 - margin.top - margin.bottom;

    this.width = width;
    this.height = height;

    // define the accession orders.
    var orders = {
      alpha_asc: "alphabetical",
      alpha_desc: "alphabetical reversed",
      phylo: "phylogeny",
      phylo_rev: "phylogeny reversed",
      ref_first: "reference accessions first"
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

    var svg = d3
      .select("#msa_chart")
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
<style scoped>
#selectButtonMSA {
  margin-left: 10px;
}

#selectMSA {
  display: inline;
  float: left;
  margin-left: 20px;
  margin-top: 20px;
  font-weight: 700;
}
</style>
