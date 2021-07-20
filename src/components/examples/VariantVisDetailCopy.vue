<template>
  <h4>Sequence Variants</h4>
  <!-- <h5>{{ msg }}</h5> -->
  <div id="chart"></div>
  <!-- <div id="dataset-picker"></div> -->
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantVisDetailTSV",
  data() {
    return {
      msg: "Hi from sequene variants component",
    };
  },
  methods: {
    updateVis(data) {
      console.log("data from update!", data);
      console.log("data.value", data[0].value);
      console.log("this.width", this.width)
      console.log("this.cellSize", this.cellSize)
      var cell = this.cellSize
      console.log("cell", cell)
      var legend_width = this.legendElementWidth
      
      // for (const { day, hour, value } of data) {
      //   console.log("TSV from update", day, hour, value);
      // }
      // data.forEach(function(d) {
      //   console.log('for each', {day: +d.day, hour: +d.hour, value: +d.value});
      //   return {
      //     day: +d.day,
      //     hour: +d.hour,
      //     value: +d.value,
      //   };
      // });

      // eslint-disable-next-line no-unused-vars
      var colorScale = d3
        .scaleQuantile()
        .domain([
          0,
          this.buckets -1,
          d3.max(data, function(d) {
            return d.value;
          }),
        ])
        .range(this.colors);

      // eslint-disable-next-line no-unused-vars
      var cards = this.svg.selectAll(".hour").data(data, function(d) {
        return d.day + ":" + d.hour;
      });

      cards.append("title");

      cards
        .enter()
        .append("rect")
        .attr("x", function(d) {
          console.log("d", d.hour, cell)
          return (d.hour - 1) * cell;
        })
        .attr("y", function(d) {
          return (d.day - 1) * cell;
        })
        .attr("rx", 4)
        .attr("ry", 4)
        .attr("class", "hour bordered")
        .attr("width", this.cellSize)
        .attr("height", this.cellSize)
        .style("fill", function(d) {
          return colorScale(d.value);
        });

      // cards
      //   .style("fill", function(d) {
      //     return colorScale(d.value);
      //   });

      cards.select("title").text(function(d) {
        return d.value;
      });

      // cards.exit().remove();

      var legend = this.svg
        .selectAll(".legend")
        .data([0].concat(colorScale.quantiles()), function(d) {
          console.log("d legend", d)
          return d;
        });

       console.log("colorScale.quantiles()", colorScale.quantiles()) //???

      legend
        .enter()
        .append("g")
        .attr("class", "legend");

      console.log('Hello?');

      legend
        .append("rect")
        .attr("x", function(d, i) {
          console.log("this.legendElementWidth?")
          return legend_width * i;
        })
        .attr("y", this.height)
        .attr("width", this.legendElementWidth)
        .attr("height", this.cellSize / 2)
        .style("fill", function(d, i) {
          return this.colors[i];
        });

      legend
        .append("text")
        .attr("class", "mono")
        .text(function(d) {
          return "≥ " + Math.round(d);
        })
        .attr("x", function(d, i) {
          return legend_width * i;
        })
        .attr("y", this.height + cell);

      // legend.exit().remove();
    },
  },
  mounted() {
    var margin = { top: 50, right: 0, bottom: 100, left: 30 },
      width = 760 - margin.left - margin.right,
      height = 430 - margin.top - margin.bottom,
      cellSize = Math.floor(width / 24),
      legendElementWidth = cellSize * 2,
      buckets = 9,
      colors = [
        "#ffffd9",
        "#edf8b1",
        "#c7e9b4",
        "#7fcdbb",
        "#41b6c4",
        "#1d91c0",
        "#225ea8",
        "#253494",
        "#081d58",
      ], // alternatively colorbrewer.YlGnBu[9]
      days = ["Mo", "Tu", "We", "Th", "Fr", "Sa", "Su"],
      times = [
        "1a",
        "2a",
        "3a",
        "4a",
        "5a",
        "6a",
        "7a",
        "8a",
        "9a",
        "10a",
        "11a",
        "12p",
        "1p",
        "2p",
        "3p",
        "4p",
        "5p",
        "6p",
        "7p",
        "8p",
        "9p",
        "10p",
        "11p",
        "12a",
      ];
      // datasets = ["data.tsv", "data2.tsv"];

    this.width = width;
    this.height = height;
    this.colors = colors;
    this.buckets = buckets;
    this.cellSize = cellSize;
    this.legendElementWidth = legendElementWidth;
  

    var svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    this.svg = svg;

    // eslint-disable-next-line no-unused-vars
    var dayLabels = svg
      .selectAll(".dayLabel")
      .data(days)
      .enter()
      .append("text")
      .text(function(d) {
        return d;
      })
      .attr("x", 0)
      .attr("y", function(d, i) {
        return i * cellSize;
      })
      .style("text-anchor", "end")
      .attr("transform", "translate(-6," + cellSize / 1.5 + ")")
      .attr("class", function(d, i) {
        return i >= 0 && i <= 4
          ? "dayLabel mono axis axis-workweek"
          : "dayLabel mono axis";
      });

    // eslint-disable-next-line no-unused-vars
    var timeLabels = svg
      .selectAll(".timeLabel")
      .data(times)
      .enter()
      .append("text")
      .text(function(d) {
        return d;
      })
      .attr("x", function(d, i) {
        return i * cellSize;
      })
      .attr("y", 0)
      .style("text-anchor", "middle")
      .attr("transform", "translate(" + cellSize / 2 + ", -6)")
      .attr("class", function(d, i) {
        return i >= 7 && i <= 16
          ? "timeLabel mono axis axis-worktime"
          : "timeLabel mono axis";
      });

    // var datasetpicker = d3
    //   .select("#dataset-picker")
    //   .selectAll(".dataset-button")
    //   .data(datasets);

    // datasetpicker
    //   .enter()
    //   .append("input")
    //   .attr("value", function(d) {
    //     return "Dataset " + d;
    //   })
    //   .attr("type", "button")
    //   .attr("class", "dataset-button")
    //   .on("click", function(d) {
    //     heatmapChart(d);
    //   });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
rect.bordered {
  stroke: #e6e6e6;
  stroke-width: 2px;
}

text.mono {
  font-size: 9pt;
  font-family: Consolas, courier;
  fill: #aaa;
}

text.axis-workweek {
  fill: #000;
}

text.axis-worktime {
  fill: #000;
}
</style>
