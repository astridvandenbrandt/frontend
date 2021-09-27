<template>
  <div id="matrix_chart"></div>
</template>

<script>
/* eslint-disable no-mixed-spaces-and-tabs */
/* eslint-disable no-unused-vars */
import * as d3 from "d3";
export default {
  name: "matrixDrag",
  mounted() {

    // // var data = [[3,5],[9,2]];
    var data_original = {
          pos: [1,2,3,4,1,2,3,4],
          base: ['a', 'b', 'a', 'b', 'b', 'b', 'a', 'b'],
          accession: ['1_Col-0', '1_Col-0', '1_Col-0', '1_Col-0', '2_An-1', '2_An-1', '2_An-1', '2_An-1'],
        };

    console.log('data matrix gene', data_original)


    function chunk(arr, size) {
  var newArr = [];
  var i;
  for (i=0; i<arr.length; i+=0) { 
      var sliced = arr.slice(i, size);
      newArr.push(sliced);
      arr.splice(0, size);
  }
  return newArr;
}

const newArr = chunk(data_original.base, 4)
console.log('nd array test', newArr)

    // Assign a 2d array of correlating values.
// This each subarray will render as a row
// const data = [
//   [1, 1, 1, 1],
//   [1, 0.8, 1, 0.5],
//   [0, 1, 1, 1],
//   [1, 1, 1, 0],
// ];

const data = [
  ['a', 'b', 'a', 'b'],
  ['a', 'b', 'a', 'b'],
];

// Add our labels as an array of strings
const columnLabelsData = ["1", "2", "3", "4"];
const rowLabelsData = [
  "1_Col-0",
  "2_An-1"

];
// const rowLabelsData = ["First Row", "Second Row", "Third Row", "Fourth Row"];
// const columnLabelsData = [
//   "First Column",
//   "Second Column",
//   "Third Column",
//   "Fourth Column",
// ];


  // Set some base properties.
  // Some come from an options object
  // pass when `Matrix` is called.
  const margin = {top: 20, right: 20, bottom: 100, left: 70},
        width = 200 - margin.left - margin.right,
        height = 200 - margin.top - margin.bottom;
    // container = options.container,
    // const startColor ="#FC7C89";
    // const endColor = "#21A38B";

  // Find our max and min values
  const maxValue = d3.max(data, (layer) => {
    return d3.max(layer, (d) => {
      return d;
    });
  });
  const minValue = d3.min(data, (layer) => {
    return d3.min(layer, (d) => {
      return d;
    });
  });

  const numrows = data.length;
  // assume all subarrays have same length
  const numcols = data[0].length;

  // Create the SVG container
  const svg = d3
    .select("#matrix_chart")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

//   // Add a background to the SVG
//   const background = svg
//     .append("rect")
//     .style("stroke", "black")
//     .attr("width", width)
//     .attr("height", height);

  // Build some scales for us to use
  const x = d3.scaleBand()
      .range([0, width]).domain(d3.range(numcols));

  const y = d3.scaleBand()
      .range([0, height])
    .domain(d3.range(numrows));

  // This scale in particular will
  // scale our colors from the start
  // color to the end color.
//   const colorMap = d3.scaleLinear()
//     .domain([minValue, maxValue])
//     .range([startColor, endColor]);

    var colors = {
      'a': "#4daf4a",
      'b': "#e41a1c",}

  // Generate rows and columns and add
  // color fills.
  const row = svg
    .selectAll(".row")
    .data(data)
    .enter()
    .append("g")
    .attr("class", "row")
    .attr("transform", (d, i) => {
      return "translate(0," + y(i) + ")";
    });

  const cell = row
    .selectAll(".cell")
    .data((d) => {
      return d;
    })
    .enter()
    .append("g")
    .attr("class", "cell")
    .attr("transform", (d, i) => {
      return "translate(" + x(i) + ", 0)";
    });

  cell
    .append("rect")
    .attr("width", x.bandwidth() - 0.3)
    .attr("height", y.bandwidth() - 0.3)
    .style("stroke-width", 4)
    .style("stroke", "white")
    .attr("rx", 4)
    .attr("ry", 4)


  row
    .selectAll(".cell")
    .data((d, i) => {
      return data[i];
    })
    // .style("fill", colorMap);
    .style("fill", (d) => colors[d])
   

  const labels = svg.append("g").attr("class", "labels");

  const columnLabels = labels
    .selectAll(".column-label")
    .data(columnLabelsData)
    .enter()
    .append("g")
    .attr("class", "column-label")
    .attr("transform", (d, i) => {
      return "translate(" + x(i) + "," + height + ")";
    });

  columnLabels
    .append("line")
    .style("stroke", "black")
    .style("stroke-width", "1px")
    .attr("x1", x.bandwidth() / 2)
    .attr("x2", x.bandwidth() / 2)
    .attr("y1", 0)
    .attr("y2", 5);

  columnLabels
    .append("text")
    .attr("x", 0)
    .attr("y", y.bandwidth() / 2 + 20)
    .attr("dy", ".82em")
    .attr("text-anchor", "end")
    .attr("transform", "rotate(-60)")
    .text((d, i) => {
      return d;
    });

  const rowLabels = labels
    .selectAll(".row-label")
    .data(rowLabelsData)
    .enter()
    .append("g")
    .attr("class", "row-label")
    .attr("transform", (d, i) => {
      return "translate(" + 0 + "," + y(i) + ")";
    });

  rowLabels
    .append("line")
    .style("stroke", "black")
    .style("stroke-width", "1px")
    .attr("x1", 0)
    .attr("x2", -5)
    .attr("y1", y.bandwidth() / 2)
    .attr("y2", y.bandwidth() / 2);

  rowLabels
    .append("text")
    .attr("x", -8)
    .attr("y", y.bandwidth() / 2)
    .attr("dy", ".32em")
    .attr("text-anchor", "end")
    .text((d, i) => {
      return d;
    });

  },
};
</script>

<style scoped></style>
