<template>
  <h4>Phylogenetic Tree</h4>
  <label id="show-length"> <input type="checkbox" /> Show branch length </label>
  <div id="phylo"></div>
</template>

<script>
import * as d3 from "d3";
import tree from "raw-loader!../assets/tree.txt";
// console.log("tree txt", tree);

export default {
  name: "TreeVis",
  mounted() {
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

    const data = parseNewick(tree);
    // console.log("tree data parsed", data);

    var rootCopy = d3
      .hierarchy(data, function(d) {
        return d.branchset;
      })
      .sum(function(d) {
        return d.branchset ? 0 : 1;
      })
      .sort(function(a, b) {
        return a.value - b.value || d3.ascending(a.data.length, b.data.length);
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

    // Margin object with properties for the four directions
    const margin = { top: 15, right: 400, bottom: 10, left: 30 }; //compute max label length for margin right

    // Width and height as the inner dimensions of the chart area
    const width = 700 - margin.left - margin.right,
      height = 280 - margin.top - margin.bottom;

    // Chart svg container
    var svg = d3
      .select("#phylo")
      .append("svg")
      .attr("class", "chart")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom);

    var g = svg
      .append("g")
      .attr("transform", `translate(${margin.left}, ${margin.top})`);

    // Cluster
    var cluster = d3.cluster().size([height, width]);

    // Initialize linear and ordinal scales (input domain and output range)
    let xScale = d3
      .scaleLinear()
      .domain([0, maxLen])
      .range([0, width]);

    // Initialize axes
    var xAxis = d3
      .axisTop(xScale)
      .ticks(6)
      .tickSizeOuter(0)
      .tickSizeInner(-height); //change 6 for more ticks

    // Draw the axis (move xAxis to the bottom with 'translate')
    // eslint-disable-next-line no-unused-vars
    var xAxisGroup = svg
      .append("g")
      .attr("class", "axis x-axis")
      .attr("transform", `translate(${margin.left}, ${margin.top + 10})`)
      .call(xAxis);

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

    // Set initial horizontal cluster
    // eslint-disable-next-line no-unused-vars
    var link = g
      .selectAll(".link")
      .data(cluster(root).links())
      .enter()
      .append("path")
      .each(function(d) {
        d.target.linkNode = this;
      })
      .attr("class", "links")
      .attr("fill", "none")
      .attr("stroke", "#ccc")
      .attr("d", function(d) {
        return horizontalStraightBranch(d);
      });

    // eslint-disable-next-line no-unused-vars
    var node = g
      .selectAll(".node")
      .data(root.descendants())
      .enter()
      .append("g")
      .attr("class", function(d) {
        return "nodes" + (d.children ? " node--internal" : " node--leaf");
      })
      .attr("transform", function(d) {
        // console.log("d.rootdist x xScale", xScale(d.rootDist));
        return "translate(" + xScale(d.rootDist) + "," + d.x + ")";
      });

    // Append circle to nodes.
    node
      .append("circle")
      .attr("fill", (d) => (d.children ? "none" : "#999"))
      .attr("r", 2.5);

    // Append text label to every leaf node.
    var leafNodeG = g.selectAll(".node--leaf");

    leafNodeG
      .append("text")
      .attr("class", "labels")
      .style("text-anchor", "start")
      .text(function(d) {
        // console.log("d text", d);
        if (d.data.name.length < 28) {
          return d.data.name.slice(0, -1); //+ " (" + d.data.length + ")";}
        } else {
          return d.data.name.slice(0, -2); //+ " (" + d.data.length + ")";
        }
      })
      // .attr("transform", "translate(" + 10 + "," + 4 + ")")
      .attr("transform", function(d) {
        return "translate(" + (width + 10 - xScale(d.rootDist)) + "," + 2 + ")";
      });

    // Append rectangle to show metdata
    leafNodeG
      .append("rect")
      .style("fill", "grey")
      .attr("width", 20)
      .attr("height", 20)
      // .attr("transform", "translate(" + 150 + "," + -11 + ")");
      .attr("transform", function(d) {
        return (
          "translate(" + (width + 140 - xScale(d.rootDist)) + "," + -11 + ")"
        );
      });

    function horizontalStraightBranch(d) {
      var array = [d.source, d.target];

      let line = d3
        .line()
        .x(function(array) {
          array.rootDist =
            (array.parent ? array.parent.rootDist : 0) +
            (array.data.length || 0);
          // console.log("array rootdist", array.data.name, array.rootDist, xScale(array.rootDist))
          return xScale(array.rootDist); //change width to scale
        })
        .y(function(array) {
          return array.x; // inverted x/y as you have a horizontal tree
        })
        .curve(d3.curveStepBefore);

      return line(array);
    }

    function dist2(a, b) {
      return (a[0] - b[0]) ** 2 + (a[1] - b[1]) ** 2;
    }

    g.on("mousemove", function() {
      const m = d3.pointer(event);
      const leaf = node.data()[
        d3.scan(node.data().map((d) => dist2([d.y, d.x], m)))
      ];

      let d = root
        .links()
        .filter((d) => d.target === leaf)
        .pop().target;
      const path = [];

      do {
        path.push(d.data);
      } while ((d = d.parent));

      // [fix] highlight subpath?
      node.classed("highlight", (d) => path.indexOf(d.data) > -1);
      node.classed("leaf", (d) => path.indexOf(d.data) === 0);
      link.classed("highlight", (d) => path.indexOf(d.target.data) > -1);

      svg.node().value = path;
      svg.node().dispatchEvent(new CustomEvent("input"));
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
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
  font-size: 9px;
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
