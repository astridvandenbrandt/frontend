<template>
  <h4>Gene Variants</h4>
  <div class="selectLabelVariants">
    <label for="selectButtonGene"> Select Gene: </label>
    <select class="selectButtonVariants" id="selectButtonGene"
      ><option value="" selected disabled hidden>gene ID</option></select
    >
    <label for="selectButtonMSA"> Sort By: </label>
    <select class="selectButtonVariants" id="selectButtonMSA"></select>
    <label for="selectButtonBrush"> Size Brush: </label>
    <select class="selectButtonVariants" id="selectButtonBrush">
      <!-- <option value="2814" selected>200 positions</option> -->
    </select>
  </div>
  <div id="gene_chart"></div>

  <div id="msa_chart"></div>
</template>

<script>
/* eslint-disable no-debugger */
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

      function filter(values, test) {
        const I = [],
          n = values.length;
        for (let i = 0; i < n; ++i) {
          if (test(values[i], i, values)) {
            I.push(i);
          }
        }
        return I;
      }

      function taker(index) {
        return (values) => values.constructor.from(index, (i) => values[i]);
      }

      // DEFINING VIS
      var visContext = this.svgContext;
      var visXcontext = this.xScaleContext;
      var visYcontext = this.yScaleContext;
      var visContextBrush = this.brush;

      var visFocus = this.svgFocus;
      var visXfocus = this.xScaleFocus;
      var visYfocus = this.yScaleFocus;
      var visColors = this.colors;
      // var visOrders = this.orders;
      // var visContextBrushSizes = this.brushSizes;

      // DATA PREPROCESSING
      console.log("data context", data_mutations);
      console.log("data focus", data);

      var flat_data = {
        pos: data.map((d) => d.pos),
        base: data.map((d) => d.base),
        accession: data.map((d) => d.accession),
      };

      console.log("flat data", flat_data);

      var length_gene = flat_data.pos.length / 14 - 1;
      console.log("length gene", length_gene);

      var start = 0;
      var end = 200;

      var flat_data_slice = [
        flat_data.pos,
        flat_data.base,
        flat_data.accession,
      ].map(taker(filter(flat_data.pos, (d) => d >= start && d <= end)));

      var flat_data_slice_default = {
        pos: flat_data_slice[0],
        base: flat_data_slice[1],
        accession: flat_data_slice[2],
      };

      // const length = 1414; //length of slice (100)
      var length = 2814; //length of slice (200)
      // const length = 4214; //length of slice (300)
      // const length = 5614; //length of slice (400)

      var rows_data_slice_default = Array.from({ length }, (_, i) => ({
        pos: flat_data_slice_default.pos[i],
        base: flat_data_slice_default.base[i],
        accession: flat_data_slice_default.accession[i],
      }));

      console.log("data_rows", rows_data_slice_default);

      /// UPDATE CONTEXT VIS
      visXcontext.domain([0, length_gene]);
      visYcontext.domain([0, 50]);

      //Defines the y axis styles`
      var xAxisContext = d3
        .axisTop()
        .scale(visXcontext)
        .tickPadding(8)
        .tickValues([
          0,
          500,
          1000,
          1500,
          2000,
          2500,
          3000,
          3500,
          4000,
          length_gene,
        ]);

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
        .call(visContextBrush)
        .call(visContextBrush.move, [start, end].map(visXcontext));

      // removes handle to resize the brush
      d3.selectAll(".brush>.handle").remove();
      // removes crosshair cursor
      d3.selectAll(".brush>.overlay").remove();

      // UPDATE FOCUS BY BRUSH
      visContextBrush.on("brush", brushed);

      function brushed({ selection }) {
        console.log("selection brush", { selection });
        const rangeSelected = selection.map(visXcontext.invert, visXcontext);
        console.log(
          "range selected: ",
          // rangeSelected,
          Math.round(rangeSelected[0]),
          Math.round(rangeSelected[1])
        );

        var start = Math.round(rangeSelected[0]);
        var end = Math.round(rangeSelected[1]) + 1;
        // console.log("start and end of brush: ", start, end);

        visXfocus.domain(d3.range(start, end));

        //rows data updated
        var flat_data_slice_updated = [
          flat_data.pos,
          flat_data.base,
          flat_data.accession,
        ].map(taker(filter(flat_data.pos, (d) => d >= start && d <= end)));

        var flat_data_slice_updated_final = {
          pos: flat_data_slice_updated[0],
          base: flat_data_slice_updated[1],
          accession: flat_data_slice_updated[2],
        };

        var length = (end - start) * 14;
        // console.log("brush length", length);

        var rows_data_slice_updated = Array.from({ length }, (_, i) => ({
          pos: parseFloat(flat_data_slice_updated_final.pos[i]), // position type should be changed from string to float
          base: flat_data_slice_updated_final.base[i],
          accession: flat_data_slice_updated_final.accession[i],
        }));
        console.log("rows updated", rows_data_slice_updated);

        xAxisFocus = d3
          .axisTop(visXfocus)
          .tickSize(10)
          .tickValues(
            visXfocus.domain().filter(function(d, i) {
              return !(i % 20);
            })
          );

        visFocus
          .selectAll(".x-axis--focus")
          // .transition()
          // .duration(300)
          .call(xAxisFocus);
        visFocus.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line
        // .on("start", function() {
        //   visFocus.select(".x-axis--focus .domain").remove();
        // });

        var visCellsUpdate = visFocus
          .selectAll(".cell")
          .data(rows_data_slice_updated);

        var visCellsEnter = visCellsUpdate
          .enter()
          .append("rect")
          .attr("class", "cell");

        visCellsUpdate.exit().remove();

        visCellsEnter
          .merge(visCellsUpdate)
          // .transition()
          //     .duration(800)
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
            return visColors[d.base];
          })
          .style("stroke-width", 4)
          .style("stroke", "none")
          .style("opacity", 0.8);


        visFocus
          .selectAll(".cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      /// UPDATE FOCUS VIS
      // Create labels for gene positions
      // var genePositions = d3
      //   .map(data, function(d) {
      //     return d.pos;
      //   })
      //   .filter(unique);
      var genePositions = flat_data_slice_default.pos.filter(unique);
      console.log("genePositions", genePositions);

      // Add domains and build axis
      visXfocus.domain(genePositions);

      var xAxisFocus = d3
        .axisTop(visXfocus)
        .tickSize(10)
        .tickValues(
          visXfocus.domain().filter(function(d, i) {
            return !(i % 20);
          })
        ); //show every fifth position

      var sortingOptions = {
        alpha_asc: flat_data.accession.filter(unique).sort(d3.descending),
        alpha_desc: flat_data.accession.filter(unique).sort(d3.ascending),
        phylo: [
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
        ],
        phylo_rev: [
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
        ].reverse(),
        ref_first: [
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
        ],
      };
      // console.log("sorting options", sortingOptions);
      //Default sorting rows
      // visYfocus.domain(
      //   d3
      //     .map(data, function(d) {
      //       return d.accession;
      //     })
      //     .filter(unique)
      //     .sort(d3.descending)
      // );
      visYfocus.domain(sortingOptions["alpha_asc"]); //default sorting
      console.log(
        "sorting options default: alpha_asc",
        sortingOptions["alpha_asc"]
      );

      // create a tooltip
      var tooltip = d3
        .select("#msa_chart")
        .append("div")
        .attr("class", "tooltip")
        .style("background-color", "black")
        .style("opacity", 0)
        .style("border", "solid")
        .style("border-width", "0px")
        .style("border-radius", "3px")
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
        // .data(flat_data_slice_final, function(d) {
        //   console.log(d)
        //   return d.pos + ":" + d.accesion;
        // })
        .data(rows_data_slice_default)
        .enter()
        .append("rect")
        .attr("class", "cell")
        .attr("x", function(d) {
          // debugger
          // console.log('d.pos', d.pos, 'd.pos xScale', visXfocus(d.pos), visXfocus.domain())
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
          return visColors[d.base];
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
        tooltip.style("opacity", 0.6).style("color", "white");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "1px")
          .style("opacity", 1);
      };

      var mousemove = function(event, d) {
        tooltip
          .html(
            "<strong>base:</strong> " +
              d.base +
              "<br/>" +
              "<strong>position:</strong> " +
              d.pos
          )
          // .html("base: " + d.base)
          .style("left", d3.pointer(event)[0] + 65 + "px")
          .style("top", d3.pointer(event)[1] + 150 + "px");
      };
      var mouseleave = function() {
        tooltip.style("opacity", 0);
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      // function to update the MSA ordering
      function updateMSAorder(sortingOption) {
        console.log("option chosen: ", sortingOption);
        visYfocus.domain(sortingOption);
        visFocus
          .selectAll(".y-axis--focus")
          .transition()
          .duration(800)
          .call(d3.axisLeft(visYfocus).tickSize(0))
          .on("start", function() {
            visFocus.select(".y-axis--focus .domain").remove();
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

      // Change row ordering based on select
      d3.select("#selectButtonMSA").on("change", function() {
        var selected = d3.select("#selectButtonMSA").node().value;
        // console.log("selected order MSA: ", selected);

        if (selected === "alpha_asc") {
          updateMSAorder(sortingOptions["alpha_asc"]);
        }
        if (selected === "alpha_desc") {
          updateMSAorder(sortingOptions["alpha_desc"]);
        }
        if (selected === "phylo") {
          updateMSAorder(sortingOptions["phylo"]);
        }
        if (selected === "phylo_rev") {
          updateMSAorder(sortingOptions["phylo_rev"]);
        }

        if (selected === "ref_first") {
          updateMSAorder(sortingOptions["ref_first"]);
        }
      });

      // Change brush size based on select
      d3.select("#selectButtonBrush").on("change", function() {
        var selectedBrush = d3.select("#selectButtonBrush").node().value;
        console.log("selected brush size: ", selectedBrush);

        var start_updated = 0;
        var end_updated = parseInt(selectedBrush) * 100;

        visContext
          .selectAll(".brush")
          .call(
            visContextBrush.move,
            [start_updated, end_updated].map(visXcontext)
          );
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

    // build color scale categories:
    // https://colorbrewer2.org/#type=qualitative&scheme=Set1&n=9
    // http://www.jalview.org/help/html/colourSchemes/ (following jalview convention)
    var colors = {
      A: "#4daf4a",
      a: "#4daf4a",
      G: "#e41a1c",
      g: "#e41a1c",
      C: "#ff7f00",
      c: "#ff7f00",
      T: "#377eb8",
      t: "#377eb8",
      "-": "lightgray",
      "*": "#999999",
    };
    this.colors = colors;

    // legendVariants labels
    var dataLabels = [1, 2, 3, 4, 5, 6];
    var cols = [
      "#4daf4a",
      "#ff7f00",
      "#e41a1c",
      "#377eb8",
      "lightgray",
      "#999999",
    ];
    var bases = ["A", "C", "G", "T", "-", "*"];

    // define the accession orders
    var orders = {
      alpha_asc: "alphabetical",
      alpha_desc: "alphabetical reversed",
      phylo: "phylogeny",
      phylo_rev: "phylogeny reversed",
      ref_first: "reference accessions first",
    };

    this.orders = orders;

    // define brush sizes
    var brushSizes = {
      1: "100 positions",
      2: "200 positions",
      3: "300 positions",
      4: "400 positions",
    };
    this.brushSizes = brushSizes;

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

    // add the options to the button
    d3.select("#selectButtonBrush")
      .selectAll("myOptionsBrush")
      .data(Object.keys(brushSizes))
      .enter()
      .append("option")
      .text(function(d) {
        return brushSizes[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value returned by the button

    // set default option
    d3.select("#selectButtonBrush").property("value", 2);

    console.log("brush sizes", Object.keys(brushSizes));

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
      .attr("class", "background-gene--context")
      .attr("width", width)
      .attr("height", focusHeight)
      .attr("transform", "translate(0," + margin.top + ")");

    var brush = d3.brushX().extent([
      [0, 0],
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

    var legendVariants = svgFocus
      .selectAll(".legendVariants")
      .data(dataLabels)
      .enter()
      .append("g")
      .attr("class", "legendVariants")
      .attr("transform", function(d, i) {
        return "translate(" + 50 * i + "," + 400 + ")";
      });

    legendVariants
      .append("rect")
      .attr("width", 10)
      .attr("height", 20)
      .attr("rx", 2)
      .attr("ry", 2)
      .style("fill", function(d, i) {
        return cols[i];
      });

    legendVariants
      .append("text")
      .attr("x", 16)
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
.selectButtonVariants {
  margin-left: 5px;
  margin-right: 20px;
}

.selectLabelVariants {
  display: inline;
  float: left;
  margin-left: 80px;
  margin-top: 20px;
  font-weight: 700;
}

.background-gene--context {
  fill: grey;
  opacity: 0.2;
}

.percentline {
  stroke: black;
  stroke-width: 1;
  opacity: 0.8;
}

.selection {
  stroke: white;
  fill: #ffff33;
  fill-opacity: 0.3;
}
</style>
