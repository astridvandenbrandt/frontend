<template>
  <h4>Sequence Variants</h4>
  <div id="msa">
    <div id="tooltip"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantVisDetail",
  mounted() {
    // create MSA plot
    var data_msa = [
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //1
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //4
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //7
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //3
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //6
      "tgtccccaaacatg---------aaattgcaaagccgttg", //5
      "tgtccccaaacatg---------aaattgcaaagccgttg", //8
      "tctccccaaacatgaaagttttcaaattgcaaagccgttg", //2
      "-G--------------------G-----------------", //Kas-1_1
      "-C-----------C--------------------------", //Sku-30_5
      "-C--------------------------------------", //Ler-0_5
      "----------------------------------------", //Gro-3_5
      "-C--------------------------------------", //Altai-5_8
      "-C--------------------------------------", //Tsu-0_8
    ];

    var dataset_msa = [];
    for (var i = 0; i < data_msa.length; i++) {
      var x = data_msa[i];
      for (var j = 0; j < x.length; j++) {
        var entry = { base: x[j], row: j, col: i };
        dataset_msa.push(entry);
      }
    }
    console.log("dataset_msa", dataset_msa);

    var margin_msa = { top: 30, right: 10, bottom: 70, left: 50 },
      width_msa = 800 - margin_msa.left - margin_msa.right,
      height_msa = 280 - margin_msa.top - margin_msa.bottom;

    var gridSize = 16;

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

    var dataLabels = [1, 2, 3, 4];
    var cols = ["#e78ac3", "#8da0cb", "#fc8d62", "#66c2a5"];
    var bases = ["A", "C", "G", "T"];

    var svg_msa = d3
      .select("#msa")
      .append("svg")
      .attr("width", width_msa + margin_msa.left + margin_msa.right)
      .attr("height", height_msa + margin_msa.top + margin_msa.bottom)
      .append("g")
      .attr(
        "transform",
        "translate(" + margin_msa.left + "," + margin_msa.top + ")"
      );

    // Initialize linear and ordinal scales (input domain and output range)
    let xScale_msa = d3
      .scaleLinear()
      // .domain([0, maxlength])
      .domain([3760, 3800])
      .range([0, width_msa - margin_msa.left - margin_msa.right - 40]);
    // .range([0, width_msa- margin_msa.right]);
    // console.log("maxlength after scale", maxlength) //weird flow in this block.. (review)

    // Initialize axes
    var xAxis_msa = d3.axisTop(xScale_msa).ticks(4); //change 6 for more ticks

    // Draw the axis (move xAxis to the bottom with 'translate')
    // eslint-disable-next-line no-unused-vars
    var xAxisGroup_msa = svg_msa
      .append("g")
      .attr("class", "axis x-axis")
      .attr("transform", `translate(${0}, ${-5})`)
      .call(xAxis_msa);

    // var tooltip = d3
    //   .select("body")
    //   .append("div")
    //   .attr("class", "tooltip")
    //   .style("opacity", 0);

    svg_msa
      .selectAll("point")
      .data(dataset_msa)
      .enter()
      .append("rect")
      .attr("class", "rect bordered")
      .attr("x", function(d) {
        return d.row * gridSize;
      })
      .attr("y", function(d) {
        return d.col * gridSize;
      })
      .attr("width", gridSize)
      .attr("height", gridSize)
      .style("fill", function(d) {
        return colors[d.base];
      })

      // Tooltip event listeners
      .on("mouseover", (event, d) => {
        d3.select("#tooltip")
          .style("opacity", 1)
          // Format number with million and thousand separator
          .html(
            `<div class="tooltip-label">Population</div>${d3.format(",")(
              d.col
            )}`
          );
      })
          // eslint-disable-next-line no-unused-vars
      .on("mousemove", (event) => {
        d3.select("#tooltip")
          .style("left", 5 + "px")
          .style("top", 5 + "px");
      })
      .on("mouseleave", () => {
        d3.select("#tooltip").style("opacity", 0);
      });

    var lbls_msa = [
      "1_",
      "4_",
      "7_",
      "3_",
      "6_",
      "5_",
      "8_",
      "2_",
      "Kas-1_1",
      "Sku-30_5",
      "Ler-0_5",
      "Gro-3_5",
      "Altai-5_8",
      "Tsu-0_8",
    ];

    // eslint-disable-next-line no-unused-vars
    var sampleLabels = svg_msa
      .selectAll(".sampleLabel")
      .data(data_msa)
      .enter()
      .append("text")
      .attr("class", "labels")
      .text(function(d, i) {
        return lbls_msa[i];
      })
      .attr("x", 0)
      .attr("y", function(d, i) {
        return i * gridSize;
      })
      .style("text-anchor", "end")
      .attr("transform", "translate(-6," + gridSize + ")");

    var legend = svg_msa
      .selectAll(".legend")
      .data(dataLabels)
      .enter()
      .append("g")
      .attr("class", "legend")
      .attr("transform", function(d, i) {
        return "translate(" + 660 + "," + (gridSize + i * gridSize) + ")";
      });

    legend
      .append("rect")
      .attr("class", "bordered")
      .attr("width", gridSize)
      .attr("height", gridSize)
      .style("fill", function(d, i) {
        return cols[i];
      });

    legend
      .append("text")
      .attr("class", "labels")
      .attr("x", 26)
      .attr("y", 10)
      .attr("dy", ".35em")
      .text(function(d, i) {
        return bases[i];
      });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
rect.bordered {
  stroke: white;
  stroke-width: 2px;
}

#tooltip {
  position: absolute;
  opacity: 0;
  background: #fff;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid #ddd;
  font-size: 12px;
  font-weight: 600;
  padding: 2px 8px;
}
.tooltip-label {
  font-weight: 500;
  font-size: 10px;
  color: #888;
}
</style>
