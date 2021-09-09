<template>
  <!-- <h4>Phylogenetic Tree</h4> -->
  <div class="selectLabelTree">
    <label id="show-length">
      <input type="checkbox" id="showBranchLength" /> Show branch length
    </label>
  </div>
  <div id="phylo"></div>
</template>

<script>
/* eslint-disable no-debugger */
import * as d3 from "d3";

export default {
  name: "TreeVis",
  methods: {
    updateVis(data_tree) {
      // console.log("raw tree data from App: ", data_tree)

      // REMOVE ELEMENTS
      d3.select("#phylo")
        .selectAll("*")
        .remove();

      // BEGIN COPIED FROM MOUNTED 
      const margin = { top: 15, right: 100, bottom: 15, left: 30 }; //compute max label length for margin right
      const width =
          d3.select("#phylo").node().clientWidth - margin.left - margin.right,
        height = 300 - margin.top - margin.bottom;
      this.margin = margin;
      this.width = width;
      this.height = height;

      // Chart svg container
      var svg = d3
        .select("#phylo")
        .append("svg")
        .attr("class", "chart")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom);

      this.svg = svg;

      var g = svg
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

      this.g = g;

      // Cluster
      var cluster = d3.cluster().size([height, width]);
      this.cluster = cluster;

      // Initialize scale
      let xScale = d3
        .scaleLinear()
        // .domain([0, maxLen])
        .range([0, width]);
      this.xScale = xScale;
      // END COPIED FROM MOUNTED 

      function parseNewick(a) {
        for (
          var e = [], r = {}, s = a.split(/\s*(;|\(|\)|,|:)\s*/), t = 0;
          t < s.length;
          t++
        ) {
          var n = s[t];
          switch (n) {
            case "(":
              var c = {};
              (r.branchset = [c]), e.push(r), (r = c);
              break;
            case ",":
              var d = {};
              e[e.length - 1].branchset.push(d), (r = d);
              break;
            case ")":
              r = e.pop();
              break;
            case ":":
              break;
            default:
              var h = s[t - 1];
              // append t to r.name in line below to deal with internal nodes of tree
              // with duplicate names. strip t from name before displaying in code below.
              ")" == h || "(" == h || "," == h
                ? (r.name = n + t)
                : ":" == h && (r.length = parseFloat(n));
          }
        }
        return r;
      }

      const data = parseNewick(data_tree);
      console.log("data tree parsed from App:", data);

      var rootCopy = d3
        .hierarchy(data, function(d) {
          return d.branchset;
        })
        .sum(function(d) {
          return d.branchset ? 0 : 1;
        })
        .sort(function(a, b) {
          return (
            a.value - b.value || d3.ascending(a.data.length, b.data.length)
          );
        });

      // compute max length of branches in tree
      var leaves = rootCopy.leaves();
      var lnghts = [];
      var i;
      var j;
      for (i = 0; i < leaves.length; i++) {
        var lts = [];
        // console.log("leaf", leaves[i].data, leaves.[i].ancestors())
        for (j = 0; j < leaves[i].ancestors().length; j++) {
          // console.log("ancestor", leaves[i].ancestors()[j].data.name, leaves[i].ancestors()[j].data.length)
          lts.push(leaves[i].ancestors()[j].data.length);
        }
        // console.log(d3.sum(lts))
        lnghts.push(d3.sum(lts));
      }
      // console.log("lengths data:", lnghts);
      // console.log("max length data:", d3.max(lnghts));
      var maxLen = d3.max(lnghts);
      console.log("maxLength branches", maxLen);

      // Sort nodes by ascending length
      var root = d3
        .hierarchy(data, function(d) {
          return d.branchset;
        })
        .sum(function(d) {
          return d.branchset ? 0 : 1;
        })
        // .sort(function(a,b) {
        //   return a.value - b.value  || d3.ascending(a.data.length, b.data.length);
        // });
        .sort(function(a, b) {
          return a.value || d3.ascending(a.data.length, b.data.length);
        });

      // DEFINING VIS
      var visTree = this.svg;
      var visG = this.g;
      var visCluster = this.cluster;
      var visX = this.xScale;

      /// UPDATE DOMAIN
      visX.domain([0, maxLen]);

      // Initialize axes
      var xAxis = d3
        .axisTop(visX)
        .ticks(6)
        .tickSizeOuter(0)
        .tickSizeInner(-this.height); //change 6 for more ticks

      // Draw the axis (move xAxis to the bottom with 'translate')
      // eslint-disable-next-line no-unused-vars
      var xAxisGroup = visTree
        .append("g")
        .attr("class", "axis x-axis")
        .attr(
          "transform",
          `translate(${this.margin.left}, ${this.margin.top + 10})`
        )
        .call(xAxis);

      // Set initial horizontal cluster
      visG
        .selectAll(".link")
        .data(visCluster(root).links())
        .enter()
        .append("path")
        // .each(function(d) {
        //   d.target.linkNode = this;
        // })
        .attr("class", "links")
        .attr("fill", "none")
        .attr("stroke", "#ccc")
        .attr("d", function(d) {   
          return horizontalStraightBranchLengths(d);
        });

      // eslint-disable-next-line no-unused-vars
      var node = visG
        .selectAll(".node")
        .data(root.descendants())
        .enter()
        .append("g")
        .attr("class", function(d) {
          return "nodes" + (d.children ? " node--internal" : " node--leaf");
        })
        .attr("transform", function(d) {
          // console.log("d.rootdist x xScale", xScale(d.rootDist));
          // return "translate(" + visX(d.rootDist) + "," + d.x + ")"; // to have label closer to branch
          return "translate(" + d.y + "," + d.x + ")";
          
        });

      // // Append circle to nodes.
      // node
      //   .append("circle")
      //   .attr("fill", (d) => (d.children ? "none" : "#999"))
      //   .attr("r", 4);

      // Append text label to every leaf node.
      var leafNodeG = visG.selectAll(".node--leaf");

      leafNodeG
        .append("text")
        .attr("class", "labels")
        .style("text-anchor", "start")
        .text(function(d) {
          console.log("d text labels", d);
          if (d.data.name.length < 7) {
            return d.data.name.slice(0, -1); //+ " (" + d.data.length + ")";}
          } else {
            return d.data.name.slice(0, -2); //+ " (" + d.data.length + ")";
          }
        })
        .attr("transform", "translate(" + 5 + "," + 5 + ")");
      // .attr("transform", function(d) {
      //   return (
      //     "translate(" + (this.width + 30 - visX(d.rootDist)) + "," + 2 + ")"
      //   );
      // });

      // // Append rectangle to show metdata
      // leafNodeG
      //   .append("rect")
      //   .style("fill", "grey")
      //   .attr("width", 20)
      //   .attr("height", 20)
      //   // .attr("transform", "translate(" + 150 + "," + -11 + ")");
      //   .attr("transform", function(d) {
      //     return (
      //       "translate(" + (this.width + 140 - visX(d.rootDist)) + "," + -11 + ")"
      //     );
      //   });

      // Show branch length option
      d3.select("#showBranchLength").on("change", function() {
        if (d3.select("#showBranchLength").property("checked")){
          console.log("selected branch length option: ", "checked");
          // visG
          // .selectAll("links")
          // .update()
          // .attr("d", function(d) {
          // return horizontalStraightBranchLengths(d);
          // });

        }
        else {
          console.log("selected branch length option: ", "unchecked");
          visG
          .selectAll("links")
          .attr("d", function(d) {
          return horizontalStraightDendrogram(d);
          });
        }


        
      });

      function horizontalStraightBranchLengths(d) {
        var array = [d.source, d.target];

        let line = d3
          .line()
          .x(function(array) {
            array.rootDist =
              (array.parent ? array.parent.rootDist : 0) +
              (array.data.length || 0);
            // console.log("array rootdist", array.data.name, array.rootDist, xScale(array.rootDist))
            return visX(array.rootDist); //change width to scale
            // return array.y; //dendrogram style
          })
          .y(function(array) {
            return array.x; // inverted x/y as you have a horizontal tree
          })
          .curve(d3.curveStepBefore);

        return line(array);
      }

      function horizontalStraightDendrogram(d) {
        var array = [d.source, d.target];

        let line = d3
          .line()
          .x(function(array) {
            array.rootDist =
              (array.parent ? array.parent.rootDist : 0) +
              (array.data.length || 0);
            // console.log("array rootdist", array.data.name, array.rootDist, xScale(array.rootDist))
            // return visX(array.rootDist); //change width to scale
            return array.y; //dendrogram style
          })
          .y(function(array) {
            return array.x; // inverted x/y as you have a horizontal tree
          })
          .curve(d3.curveStepBefore);

        return line(array);
      }
    },

    
  },
  mounted() {
    //set the dimensions and margins of the graph
    // const margin = { top: 15, right: 100, bottom: 15, left: 30 }; //compute max label length for margin right
    // const width =
    //     d3.select("#phylo").node().clientWidth - margin.left - margin.right,
    //   height = 300 - margin.top - margin.bottom;
    // this.margin = margin;
    // this.width = width;
    // this.height = height;
    // // Chart svg container
    // var svg = d3
    //   .select("#phylo")
    //   .append("svg")
    //   .attr("class", "chart")
    //   .attr("width", width + margin.left + margin.right)
    //   .attr("height", height + margin.top + margin.bottom);
    // this.svg = svg;
    // var g = svg
    //   .append("g")
    //   .attr("transform", `translate(${margin.left}, ${margin.top})`);
    // this.g = g;
    // // Cluster
    // var cluster = d3.cluster().size([height, width]);
    // this.cluster = cluster;
    // // Initialize scale
    // let xScale = d3
    //   .scaleLinear()
    //   // .domain([0, maxLen])
    //   .range([0, width]);
    // this.xScale = xScale;
    // function dist2(a, b) {
    //   return (a[0] - b[0]) ** 2 + (a[1] - b[1]) ** 2;
    // }
    // g.on("mousemove", function() {
    //   const m = d3.pointer(event);
    //   const leaf = node.data()[
    //     d3.scan(node.data().map((d) => dist2([d.y, d.x], m)))
    //   ];
    //   let d = root
    //     .links()
    //     .filter((d) => d.target === leaf)
    //     .pop().target;
    //   const path = [];
    //   do {
    //     path.push(d.data);
    //   } while ((d = d.parent));
    //   // [fix] highlight subpath?
    //   node.classed("highlight", (d) => path.indexOf(d.data) > -1);
    //   node.classed("leaf", (d) => path.indexOf(d.data) === 0);
    //   link.classed("highlight", (d) => path.indexOf(d.target.data) > -1);
    //   svg.node().value = path;
    //   svg.node().dispatchEvent(new CustomEvent("input"));
    // });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.selectLabelTree {
  display: inline;
  float: left;
  margin-left: 30px;
  margin-top: 0px;
  font-weight: 700;
}

.axis path,
.axis line {
  fill: none;
  stroke: grey;
  shape-rendering: crispEdges;
}

.axis .tick line {
  fill: none;
  stroke: lightgrey;
  stroke-dasharray: 5, 10;
  opacity: 0.7;
  shape-rendering: crispEdges;
}

.labels {
  font-size: 15px;
  font-family: sans-serif;
  fill: #2c3e50;
}

/* .label--active {
  font-weight: bold;
  font-size: 9px;
}

.link--active {
  stroke: #007bff !important;
  stroke-width: 2.5px;
  shape-rendering: crispEdges;
} */

.links {
  fill: none;
  stroke: black;
  stroke-width: 2px;
  shape-rendering: crispEdges;
}

.highlight circle {
  fill: lightblue;
}
.highlight circle {
  fill: lightblue;
}
.highlight text {
  fill: lightblue;
}
.leaf circle {
  fill: blue;
}
.leaf circle {
  fill: blue;
}
.leaf text {
  fill: blue;
}
.leaf rect {
  stroke: blue;
}
path.highlight {
  stroke: lightblue;
  shape-rendering: crispEdges;
}
</style>
