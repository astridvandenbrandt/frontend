<template>
  <h4>Sorting example</h4>
  <h5>{{ msg }}</h5>
  <!-- <button id="byKey">Sort by Key</button>
  <button id="byValue">Sort by Value</button> -->
  <select id="selectButtonBars"
    >...</select
  >
  <div id="my_chart"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "update example",
  data() {
    return {
      msg: "Hi from barchart reordering example",
    };
  },
  methods: {
    updateVis(data) {
      data.forEach(function(d) {
        d.value = Math.round(d.value * 1000) / 1000;
      });

      console.log("alphabet data", data);

      var vis = this.svg;
      var visHeight = this.height;
      var visY = this.y;
      var visX = this.x;

      visY.domain([
        0,
        d3.max(data, function(d) {
          return d.value;
        }),
      ]);

      var yAxis = d3.axisLeft().scale(visY);

      visX.domain(
        data.map(function(d) {
          return d.key;
        })
      );

      vis
        .append("g")
        .attr("class", "axis")
        .attr("transform", "translate(" + this.margin.left + ",0)")
        .call(yAxis);

      vis
        .selectAll("rect")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "bar")
        .on("mouseover", function() {
          d3.select(this).attr("fill", "blue");
        })
        .on("mouseout", function() {
          d3.select(this)
            .transition("colorfade")
            .duration(250)
            .attr("fill", function(d) {
              return (
                "rgb(" +
                Math.round(d.value * 1500) +
                "," +
                Math.round(d.value * 1500) +
                "," +
                Math.round(d.value * 1500) +
                ")"
              );
            });
        })

        .attr("fill", function(d) {
          return (
            "rgb(" +
            Math.round(d.value * 1500) +
            "," +
            Math.round(d.value * 1500) +
            "," +
            Math.round(d.value * 1500) +
            ")"
          );
        })

        .attr("x", function(d) {
          return visX(d.key);
        })
        .attr("width", visX.bandwidth())
        .attr("y", this.height)

        .transition("bars")
        .delay(function(d, i) {
          return i * 50;
        })
        .duration(500)

        .attr("y", function(d) {
          return visY(d.value);
        })
        .attr("height", function(d) {
          return visHeight - visY(d.value);
        });

      vis
        .selectAll("rect")
        .append("title")
        .text(function(d) {
          return d.key + ": " + d.value;
        });

      vis
        .selectAll(".val-label")
        .data(data)
        .enter()
        .append("text")
        .classed("val-label", true)

        .attr("x", function(d) {
          return visX(d.key) + visX.bandwidth() / 2;
        })
        .attr("y", visHeight)

        .transition("label")
        .delay(function(d, i) {
          return i * 50; // gives it a smoother effect
        })
        .duration(500)

        .attr("y", function(d) {
          return visY(d.value) - 4;
        })
        .attr("text-anchor", "middle")
        .style("font-size", "8px")
        .text(function(d) {
          return d.value;
        });

      vis
        .selectAll(".bar-label")
        .data(data)
        .enter()
        .append("text")
        .classed("bar-label", true)

        .attr("transform", function(d) {
          return (
            "translate(" +
            (visX(d.key) + visX.bandwidth() / 2 - 5) +
            "," +
            (visHeight + 15) +
            ")"
          );
        })

        .attr("text-anchor", "left")
        .text(function(d) {
          return d.key;
        });

      d3.select("#selectButtonBars").on("change", function() {
        var selected = d3.select("#selectButtonBars").node().value;
        console.log("selected: ", selected);

        if (selected === "key") {
          data.sort(function(a, b) {
            return d3.ascending(a.key, b.key);
          });
          visX.domain(
            data.map(function(d) {
              return d.key;
            })
          );
          vis
            .selectAll(".bar")
            .transition()
            .duration(500)
            .attr("x", function(d) {
              return visX(d.key);
            });

          vis
            .selectAll(".val-label")
            .transition()
            .duration(500)
            .attr("x", function(d) {
              return visX(d.key) + visX.bandwidth() / 2;
            });

          vis
            .selectAll(".bar-label")
            .transition()
            .duration(500)
            .attr("transform", function(d) {
              return (
                "translate(" +
                (visX(d.key) + visX.bandwidth() / 2 - 5) +
                "," +
                (visHeight + 15) +
                ")"
              );
            });
        } else {
          data.sort(function(a, b) {
            return d3.descending(a.value, b.value);
          });
          visX.domain(
            data.map(function(d) {
              return d.key;
            })
          );
          vis
            .selectAll(".bar")
            .transition()
            .duration(500)
            .attr("x", function(d) {
              return visX(d.key);
            });

          vis
            .selectAll(".val-label")
            .transition()
            .duration(500)
            .attr("x", function(d) {
              return visX(d.key) + visX.bandwidth() / 2;
            });

          vis
            .selectAll(".bar-label")
            .transition()
            .duration(500)
            .attr("transform", function(d) {
              return (
                "translate(" +
                (visX(d.key) + visX.bandwidth() / 2 - 5) +
                "," +
                (visHeight + 15) +
                ")"
              );
            });
        }
      });
    },
  },
  mounted() {
    var w = 700;
    var h = 500;
    var margin = {
      top: 50,
      bottom: 100,
      left: 30,
      right: 20,
    };
    var width = w - margin.left - margin.right;
    var height = h - margin.top - margin.bottom;

    this.margin = margin;
    this.width = width;
    this.height = height;

    var orders = { key: "alphabetical key", value: "descending value" };

    // add the options to the button
    d3.select("#selectButtonBars")
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

    var y = d3.scaleLinear().range([height, margin.top]);

    this.y = y;

    var x = d3
      .scaleBand()
      .range([margin.left, width])
      .padding(0.1);

    this.x = x;

    var svg = d3
      .select("#my_chart")
      .append("svg")
      .attr("id", "chart")
      .attr("width", w)
      .attr("height", h);

    this.svg = svg;
  },
};
</script>

//
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style></style>
