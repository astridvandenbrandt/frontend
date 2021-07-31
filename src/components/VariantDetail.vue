<template>
  <h4>Gene Variants</h4>
  <div id="selectGene">
    <label for="selectButtonGene"> Select Gene: </label>
    <select id="selectButtonGene"
      ><option value="" selected disabled hidden>gene ID</option></select
    >
    <label for="selectButtonMSA"> Sort By: </label>
    <select id="selectButtonMSA"></select>
  </div>
  <div id="gene_chart"></div>

  <div id="msa_chart"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "VariantDetail",
  // data() {
  // },
  methods: {
    updateVis(data, data_mutations) {
      const unique = (value, index, self) => {
        return self.indexOf(value) === index;
      };

      console.log("data mutations", data_mutations);

      var visContext = this.svgContext;
      var visXcontext = this.xScaleContext;
      var visYcontext = this.yScaleContext;

      var visFocus = this.svgFocus;
      var visXfocus = this.xScaleFocus;
      var visYfocus = this.yScaleFocus;

      
      /// UPDATE CONTEXT VIS 
      visXcontext.domain([0, 4383]);
      visYcontext.domain([0, 50]);

      //Defines the y axis styles`
      var xAxisContext = d3
        .axisTop()
        .scale(visXcontext)
        .tickPadding(8)
        .tickValues([0, 500, 1000, 1500, 2000, 2500, 3000, 3500, 4000, 4383]);

      //Appends the x axis
      visContext
        .append("g")
        .attr("class", "x-axis--context")
        .attr("transform", "translate(0," + this.margin.top + ")")
        .call(xAxisContext);

      //Binds data to strips
      visContext
        .selectAll("line.percent")
        .data(data_mutations)
        .enter()
        .append("line")
        .attr("class", "percentline")
        .attr("x1", function(d) {
          // console.log("x1", d.position, xScale(d.position));
          return visXcontext(d.position);
        })
        .attr("x2", function(d) {
          return visXcontext(d.position);
        })
        .attr("y1", 30)
        .attr("y2", 70);

      visContext
        .append("g")
        .attr("class", "brush")
        .attr("transform", "translate(0," + this.margin.top + ")")
        .call(this.brush)
        .call(this.brush.move, [3700, 3900].map(visXcontext));

      // removes handle to resize the brush
      d3.selectAll(".brush>.handle").remove();
      // removes crosshair cursor
      d3.selectAll(".brush>.overlay").remove();

      
      
      /// UPDATE FOCUS VIS 
      // Create labels for gene positions
      var genePositions = d3
        .map(data, function(d) {
          return d.pos;
        })
        .filter(unique);

      // Add domains and build axis
      visXfocus.domain(genePositions);

      var xAxisFocus = d3
        .axisTop(visXfocus)
        .tickSize(10)
        .tickValues(
          visXfocus.domain().filter(function(d, i) {
            return !(i % 25);
          })
        ); //show every fifth position

      // Default sorting rows
      visYfocus.domain(
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
        "*": "grey",
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
      visFocus
        .append("g")
        .attr("class", "x-axis--focus")
        .style("font-size", 15)
        // .attr("transform", "translate(0," + this.height + ")")
        .call(xAxisFocus)
        .select(".domain")
        .remove();

      visFocus
        .append("g")
        .attr("class", "y-axis--focus")
        .style("font-size", 15)
        .call(d3.axisLeft(visYfocus).tickSize(0))
        .select(".domain")
        .remove();

      // Add the squares
      visFocus
        .selectAll()
        .data(data, function(d) {
          return d.pos + ":" + d.accesion;
        })
        .enter()
        .append("rect")
        .attr("class", "cell")
        .attr("x", function(d) {
          return visXfocus(d.pos);
        })
        .attr("y", function(d) {
          return visYfocus(d.accession);
        })
        .attr("rx", 2)
        .attr("ry", 2)
        .attr("width", visXfocus.bandwidth())
        .attr("height", visYfocus.bandwidth())
        .style("fill", function(d) {
          return colors[d.base];
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8);

      visFocus
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
          visYfocus.domain(
            d3
              .map(data, function(d) {
                return d.accession;
              })
              .filter(unique)
              .sort(d3.descending)
          );
          visFocus
            .selectAll(".y-axis")
            .transition()
            .duration(800)
            .call(d3.axisLeft(visYfocus).tickSize(0))
            .on("start", function() {
              visFocus.select(".y-axis .domain").remove();
            });
          visFocus
            .selectAll(".cell")
            .transition()
            .duration(800)
            .attr("x", function(d) {
              return visXfocus(d.pos);
            })
            .attr("y", function(d) {
              return visYfocus(d.accession);
            });
          visFocus
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "alpha_desc") {
          // sorting rows
          visYfocus.domain(
            d3
              .map(data, function(d) {
                return d.accession;
              })
              .filter(unique)
              .sort(d3.ascending)
          );
          visFocus
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visYfocus).tickSize(0))
            .on("start", function() {
              visFocus.select(".y-axis .domain").remove();
            });
          visFocus
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visXfocus(d.pos);
            })
            .attr("y", function(d) {
              return visYfocus(d.accession);
            });
          visFocus
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "phylo") {
          // sorting rows
          visYfocus.domain([
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
          visFocus
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visYfocus).tickSize(0))
            .on("start", function() {
              visFocus.select(".y-axis .domain").remove();
            });
          visFocus
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visXfocus(d.pos);
            })
            .attr("y", function(d) {
              return visYfocus(d.accession);
            });
          visFocus
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }
        if (selected === "phylo_rev") {
          // sorting rows
          visYfocus.domain([
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
          visFocus
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visYfocus).tickSize(0))
            .on("start", function() {
              visFocus.select(".y-axis .domain").remove();
            });
          visFocus
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visXfocus(d.pos);
            })
            .attr("y", function(d) {
              return visYfocus(d.accession);
            });
          visFocus
            .selectAll(".cell")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave);
        }

        if (selected === "ref_first") {
          // sorting rows
          visYfocus.domain([
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
          visFocus
            .selectAll(".y-axis")
            .transition()
            .duration(1000)
            .call(d3.axisLeft(visYfocus).tickSize(0))
            .on("start", function() {
              visFocus.select(".y-axis .domain").remove();
            });
          visFocus
            .selectAll(".cell")
            .transition()
            .duration(1000)
            .attr("x", function(d) {
              return visXfocus(d.pos);
            })
            .attr("y", function(d) {
              return visYfocus(d.accession);
            });
          visFocus
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
    var margin = { top: 30, right: 20, bottom: 30, left: 80 },
      width =
        d3.select("#msa_chart").node().clientWidth - margin.left - margin.right,
      height = 450 - margin.top - margin.bottom;

    var focusHeight = 40;

    this.margin = margin;
    this.width = width;
    this.height = height;

    // define the accession orders.
    var orders = {
      alpha_asc: "alphabetical",
      alpha_desc: "alphabetical reversed",
      phylo: "phylogeny",
      phylo_rev: "phylogeny reversed",
      ref_first: "reference accessions first",
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

    //Creates the context xScale
    var xScaleContext = d3.scaleLinear().range([0, width]);
    this.xScaleContext = xScaleContext;

    //Creates the context yScale
    var yScaleContext = d3.scaleLinear().range([focusHeight, 0]);
    this.yScaleContext = yScaleContext;

    var svgContext = d3
      .select("#gene_chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", focusHeight + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + 0 + ")");

    this.svgContext = svgContext;

    //Appends background color
    svgContext
      .append("rect")
      .attr("class", "background-gene-context")
      .attr("width", width)
      .attr("height", focusHeight)
      .attr("transform", "translate(0," + margin.top + ")");

    var brush = d3.brushX().extent([
      [100, 0],
      [width, focusHeight],
    ]);
    this.brush = brush;

    var xScaleFocus = d3
      .scaleBand()
      .range([0, width])
      .padding(0.05);

    this.xScaleFocus = xScaleFocus;

    var yScaleFocus = d3
      .scaleBand()
      .range([height, 0])
      .padding(0.05);

    this.yScaleFocus = yScaleFocus;

    var svgFocus = d3
      .select("#msa_chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    this.svgFocus = svgFocus;
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#selectButtonGene {
  margin-left: 5px;
  margin-right: 25px;
}

#selectGene {
  display: inline;
  float: left;
  margin-left: 80px;
  margin-top: 20px;
  font-weight: 700;
}
#selectButtonMSA {
  margin-left: 5px;
  margin-right: 15px;
}

#selectMSA {
  display: inline;
  float: left;
  margin-left: 80px;
  margin-top: 20px;
  font-weight: 700;
}

.background-gene-context {
  fill: lightsteelblue;
  opacity: 0.2;
}

.percentline {
  stroke: red;
  stroke-width: 1;
  opacity: 0.8;
}
/* 
.selection {
  stroke: white;
  fill: green;
  fill-opacity: 0.165;
} */
</style>
