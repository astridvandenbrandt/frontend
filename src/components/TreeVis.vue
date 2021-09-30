<template>
  <!-- <h4>Phylogenetic Tree</h4> -->
  <div class="selectLabelTree">
    <label id="show-length">
      <input type="checkbox" id="showBranchLength" checked/> Show branch length
    </label>
  </div>
  <div id="phylo"></div>
</template>

<script>
/* eslint-disable no-debugger */
/* eslint-disable no-unused-vars */
import * as d3 from "d3";
export default {
  name: "TreeVis",
  methods: {
    updateVis(data_tree) {
      let vis = this;

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
      console.log("data tree parsed", data);

      // Create data hierarchy and sort nodes by ascending length
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
      vis.root = root;

      // compute max length of branches in tree
      var leaves = root.leaves();
      var lnghts = [];
      var i;
      var j;
      for (i = 0; i < leaves.length; i++) {
        var lts = [];
        for (j = 0; j < leaves[i].ancestors().length; j++) {
          lts.push(leaves[i].ancestors()[j].data.length);
        }
        lnghts.push(d3.sum(lts));
      }

      var maxLen = d3.max(lnghts);
      console.log("maxLength branches", maxLen);

      // Set the scale input domains
      vis.xScale.domain([0, maxLen]);

      vis.renderVis();
    },
    renderVis() {
      let vis = this;

      updateTreeChart(vis.cluster(vis.root).links(), vis.root.descendants());

      // update axis
      vis.xAxisGroup.call(vis.xAxis);

      function updateTreeChart(dataLinks, dataNodes) {

          // check new data 
        let visTreeUpdate = vis.g
          .selectAll(".links")
          .data(dataLinks);

        // remove old tree
        visTreeUpdate.exit().remove();

        // append data to links
        let visTreeEnter = visTreeUpdate
          .enter()
          .append("path")
          .attr("class", "links");
      
        // merge data 
        visTreeEnter
          .merge(visTreeUpdate)
          .transition()
          .attr("fill", "none")
          .attr("stroke", "#ccc")
          .attr("d", d => horizontalStraightBranchLengths(d));

        // check new node data 
        let visTreeNodesUpdate = vis.g
          .selectAll(".nodes")
          .data(dataNodes);

        // remove old nodes
        visTreeNodesUpdate.exit().remove();

        // append data to links
        let visTreeNodesEnter = visTreeNodesUpdate
          .enter()
          .append("circle")
          .attr("class", "nodes");

          visTreeNodesEnter
          .merge(visTreeNodesUpdate)
          .transition()
          .attr("fill", (d) => (d.children ? "none" : "#999"))
          .attr("r", 2)
          .attr("transform", d =>
            // console.log("d.rootdist x xScale", xScale(d.rootDist));
            "translate(" + vis.xScale(d.rootDist) + "," + d.x + ")" // to have label closer to branch
            //  "translate(" + d.y + "," + d.x + ")"
          );

        // check new node data 
        let visTreeLabelsUpdate = vis.g
          .selectAll(".node-labels")
          .data(dataNodes);

        // remove old nodes
        visTreeLabelsUpdate.exit().remove();

        // append data to links
        let visTreeLabelsEnter = visTreeLabelsUpdate
          .enter()
          .append("text")
          .attr("class", "node-labels");

          visTreeLabelsEnter
          .merge(visTreeLabelsUpdate)
          .transition()
          .attr("fill", (d) => (d.children ? "none" : "#2c3e50"))
          .attr("font-size", "10px")
          .attr("transform", d =>
            // console.log("d.rootdist x xScale", xScale(d.rootDist));
            "translate(" + (vis.xScale(d.rootDist) + 5) + "," + ( d.x + 2) + ")" // to have label closer to branch
            //  "translate(" + (d.y + 5) + "," +  ( d.x + 2) + ")"
          )
          .text(d => {
            if (d.data.name.length < 7) {
                return d.data.name.slice(0, -1); //+ " (" + d.data.length + ")";}
              } else {
                return d.data.name.slice(0, -2); //+ " (" + d.data.length + ")";
              }
            })
          .style("text-anchor", "start")

      }

      function updateDendrogramChart(dataLinks, dataNodes) {

        // check new data 
        let visTreeUpdate = vis.g
        .selectAll(".links")
        .data(dataLinks);

        // remove old tree
        visTreeUpdate.exit().remove();

        // append data to links
        let visTreeEnter = visTreeUpdate
        .enter()
        .append("path")
        .attr("class", "links");

        // merge data 
        visTreeEnter
        .merge(visTreeUpdate)
        .transition()
        .attr("fill", "none")
        .attr("stroke", "#ccc")
        .attr("d", d => horizontalStraightDendrogram(d));

        // check new node data 
        let visTreeNodesUpdate = vis.g
        .selectAll(".nodes")
        .data(dataNodes);

        // remove old nodes
        visTreeNodesUpdate.exit().remove();

        // append data to links
        let visTreeNodesEnter = visTreeNodesUpdate
        .enter()
        .append("circle")
        .attr("class", "nodes");

        visTreeNodesEnter
        .merge(visTreeNodesUpdate)
        .transition()
        .attr("fill", (d) => (d.children ? "none" : "#999"))
        .attr("r", 2)
        .attr("transform", d =>
          // console.log("d.rootdist x xScale", xScale(d.rootDist));
          // "translate(" + vis.xScale(d.rootDist) + "," + d.x + ")" // to have label closer to branch
           "translate(" + d.y + "," + d.x + ")"
        );

        // check new node data 
        let visTreeLabelsUpdate = vis.g
        .selectAll(".node-labels")
        .data(dataNodes);

        // remove old nodes
        visTreeLabelsUpdate.exit().remove();

        // append data to links
        let visTreeLabelsEnter = visTreeLabelsUpdate
        .enter()
        .append("text")
        .attr("class", "node-labels");

        visTreeLabelsEnter
        .merge(visTreeLabelsUpdate)
        .transition()
        .attr("fill", (d) => (d.children ? "none" : "#2c3e50"))
        .attr("font-size", "10px")
        .attr("transform", d =>
          // console.log("d.rootdist x xScale", xScale(d.rootDist));
          // "translate(" + (vis.xScale(d.rootDist) + 5) + "," + ( d.x + 2) + ")" // to have label closer to branch
           "translate(" + (d.y + 5) + "," +  ( d.x + 2) + ")"
        )
        .text(d => {
          if (d.data.name.length < 7) {
              return d.data.name.slice(0, -1); //+ " (" + d.data.length + ")";}
            } else {
              return d.data.name.slice(0, -2); //+ " (" + d.data.length + ")";
            }
          })
        // .style("text-anchor", "start")

        }

      // Show branch length option
      d3.select("#showBranchLength").on("change", function() {
        if (d3.select("#showBranchLength").property("checked")) {
          console.log("selected branch length option: ", "checked");
          updateTreeChart(vis.cluster(vis.root).links(), vis.root.descendants());
          d3.select("#x-axis--tree").style("opacity", "1");

  
        } else {
          console.log("selected branch length option: ", "unchecked");
          updateDendrogramChart(vis.cluster(vis.root).links(), vis.root.descendants());
          // vis.xAxisGroup.select(".axis x-axis").remove(); // to disable rendering the axis line
          d3.select("#x-axis--tree").style("opacity", "0");



        
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
            return vis.xScale(array.rootDist); //change width to scale
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
    let vis = this;
    //set the dimensions and margins of the graph
    const margin = { top: 15, right: 45, bottom: 15, left: 30 }; //compute max label length for margin right
    const width =
        d3.select("#phylo").node().clientWidth - margin.left - margin.right,
      height = 250 - margin.top - margin.bottom;
    vis.margin = margin;
    vis.width = width;
    vis.height = height;

    // Chart svg container
    var svg = d3
      .select("#phylo")
      .append("svg")
      .attr("class", "chart")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom);
    vis.svg = svg;

    var g = svg
      .append("g")
      .attr("transform", `translate(${margin.left}, ${margin.top})`);
    vis.g = g;

    // Cluster
    var cluster = d3.cluster().size([height, width]);
    vis.cluster = cluster;

    // Initialize scale
    let xScale = d3
      .scaleLinear()
      // .domain([0, maxLen])
      .range([0, width]);
    vis.xScale = xScale;

    // Initialize axes
    var xAxis = d3
      .axisTop(vis.xScale)
      .ticks(6)
      .tickSizeOuter(0)
      .tickSizeInner(-vis.height); //change 6 for more ticks

    vis.xAxis = xAxis;

    var xAxisGroup = vis.svg
      .append("g")
      .attr("class", "axis x-axis")
      .attr("id", "x-axis--tree")
      .attr(
        "transform",
        `translate(${vis.margin.left}, ${vis.margin.top + 10})`
      );
    vis.xAxisGroup = xAxisGroup;

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
  margin-left: 0.5rem;
  margin-top: 0.5rem;
  /* font-weight: 700; */
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
  font-size: 10px;
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
  /* shape-rendering: crispEdges; */
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
