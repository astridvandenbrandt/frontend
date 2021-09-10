<template>
  <div class="selectLabelVariants">
    <!-- <label for="selectButtonGene"> Select Gene: </label>
    <select class="selectButtonVariants" id="selectButtonGene"
      ><option value="" selected disabled hidden>gene ID</option></select
    > -->
    <label for="selectButtonSort"> Sort By: </label>
    <select class="selectButtonVariants" id="selectButtonSort"></select>
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
/* eslint-disable no-unused-vars */
import * as d3 from "d3";
export default {
  name: "GeneVariantVis",
  methods: {
    // This function contains all the code to prepare the data before we render it.
    updateVis(data, data_mutations) {
      let vis = this;

      // helper functions
      const unique = (value, index, self) => {
        return self.indexOf(value) === index;
      };
      vis.unique = unique;

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
      vis.filter = filter;

      function taker(index) {
        return (values) => values.constructor.from(index, (i) => values[i]);
      }
      vis.taker = taker;

      // data preprocessing
      vis.data_mutations = data_mutations;
      console.log("data context vis:", data_mutations);
      console.log("data focus vis: ", data);

      const flat_data = {
        pos: data.map((d) => parseFloat(d.pos)),
        base: data.map((d) => d.base),
        accession: data.map((d) => d.accession),
      };
      console.log("flat data", flat_data);
      vis.flat_data = flat_data;

      const nr_accessions = [...new Set(flat_data.accession)].length;
      console.log("nr accessions", nr_accessions);
      vis.nr_accessions = nr_accessions;

      const length_gene = data.length / nr_accessions - 1;
      vis.length_gene = length_gene;
      console.log("length gene", length_gene);

      const start = 0;
      vis.start = start;
      const updateBrush = d3.select("#selectButtonBrush").node().value;
      const end = updateBrush * 100; // when new data loaded keep brush size
      vis.end = end;
      const updateSort = d3.select("#selectButtonSort").node().value;

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

      var genePositions = flat_data_slice_default.pos.filter(unique);
      console.log("genePositions", genePositions);

      var sortingOptions = {
        alpha_asc: [
          "8__Tsu-0",
          "8__Kas-1",
          "8__Altai-5",
          "8_Sha",
          "7__Sku-30",
          "7__Ler-0",
          "7__Gro-3",
          "7_Ler",
          "6__Tsu-0",
          "6__Kas-1",
          "6__Altai-5",
          "6_Kyo",
          "5__Sku-30",
          "5__Ler-0",
          "5__Gro-3",
          "5_Eri",
          "4_Cvi",
          "3_C24",
          "2_An-1",
          "1__Tsu-0",
          "1__Sku-30",
          "1__Ler-0",
          "1__Kas-1",
          "1__Gro-3",
          "1__Altai-5",
          "1_Col-0",
        ],
        alpha_desc: [
          "8__Tsu-0",
          "8__Kas-1",
          "8__Altai-5",
          "8_Sha",
          "7__Sku-30",
          "7__Ler-0",
          "7__Gro-3",
          "7_Ler",
          "6__Tsu-0",
          "6__Kas-1",
          "6__Altai-5",
          "6_Kyo",
          "5__Sku-30",
          "5__Ler-0",
          "5__Gro-3",
          "5_Eri",
          "4_Cvi",
          "3_C24",
          "2_An-1",
          "1__Tsu-0",
          "1__Sku-30",
          "1__Ler-0",
          "1__Kas-1",
          "1__Gro-3",
          "1__Altai-5",
          "1_Col-0",
        ].reverse(),
        phylo: [
          "2_An-1",
          "1__Tsu-0",
          "1__Sku-30",
          "1__Ler-0",
          "1__Kas-1",
          "1__Gro-3",
          "1__Altai-5",
          "1_Col-0",
          "4_Cvi",
          "3_C24",
          "8__Tsu-0",
          "8__Kas-1",
          "8__Altai-5",
          "8_Sha",
          "7__Sku-30",
          "7__Ler-0",
          "7__Gro-3",
          "7_Ler",
          "5__Sku-30",
          "5__Ler-0",
          "5__Gro-3",
          "5_Eri",
          "6__Tsu-0",
          "6__Kas-1",
          "6__Altai-5",
          "6_Kyo",
        ],
        phylo_rev: [
          "2_An-1",
          "1__Tsu-0",
          "1__Sku-30",
          "1__Ler-0",
          "1__Kas-1",
          "1__Gro-3",
          "1__Altai-5",
          "1_Col-0",
          "4_Cvi",
          "3_C24",
          "8__Tsu-0",
          "8__Kas-1",
          "8__Altai-5",
          "8_Sha",
          "7__Sku-30",
          "7__Ler-0",
          "7__Gro-3",
          "7_Ler",
          "5__Sku-30",
          "5__Ler-0",
          "5__Gro-3",
          "5_Eri",
          "6__Tsu-0",
          "6__Kas-1",
          "6__Altai-5",
          "6_Kyo",
        ].reverse(),
        ref_first: [
          "8__Tsu-0",
          "8__Altai-5",
          "8__Kas-1",
          "5__Gro-3",
          "5__Ler-0",
          "5__Sku-30",
          "6__Altai-5",
          "6__Kas-1",
          "6__Tsu-0",
          "7__Sku-30",
          "7__Gro-3",
          "7__Ler-0",
          "1__Gro-3",
          "1__Ler-0",
          "1__Sku-30",
          "1__Altai-5",
          "1__Kas-1",
          "1__Tsu-0",
          "2_An-1",
          "8_Sha",
          "5_Eri",
          "6_Kyo",
          "3_C24",
          "7_Ler",
          "4_Cvi",
          "1_Col-0",
        ],
      };
      vis.sortingOptions = sortingOptions;

      // Set the scale input domains
      vis.xScaleContext.domain([0, vis.length_gene]);
      vis.yScaleContext.domain([0, 50]);

      vis.xScaleFocus.domain(genePositions);
      vis.yScaleFocus.domain(sortingOptions[updateSort]);

      var length = (end - start + 1) * 26; //length of slice (200)

      var rows_data_slice_default = Array.from({ length }, (_, i) => ({
        pos: flat_data_slice_default.pos[i],
        base: flat_data_slice_default.base[i],
        accession: flat_data_slice_default.accession[i],
      }));
      console.log("rows_data_slice_default", rows_data_slice_default);
      vis.rows_data_slice_default = rows_data_slice_default;

      vis.renderVis();
    },
    // This function contains the D3 code for binding data to visual elements.
    renderVis() {
      let vis = this;

      // console.log('data mutations new:', vis.data_mutations)
      var visGeneUpdate = vis.svgContext
        .selectAll(".variants--summary")
        .data(vis.data_mutations);

      var visGeneEnter = visGeneUpdate
        .enter()
        .append("line")
        .attr("class", "variants--summary");

      visGeneUpdate.exit().remove();

      visGeneEnter
        .merge(visGeneUpdate)
        .attr("x1", function(d) {
          return vis.xScaleContext(d.pos);
        })
        .attr("x2", function(d) {
          return vis.xScaleContext(d.pos);
        })
        .attr("y1", 30)
        .attr("y2", 60);

      vis.svgContext
        .selectAll(".brush")
        .call(vis.brush)
        .call(vis.brush.move, [vis.start, vis.end].map(vis.xScaleContext));

      // removes handle to resize the brush
      d3.selectAll(".brush>.handle").remove();
      // removes crosshair cursor
      d3.selectAll(".brush>.overlay").remove();

      // update figure when brushing
      vis.brush.on("end", brushed); //change 'end' to 'brush' if want to see inbetween

      var visCellsUpdate = vis.svgFocus
        .selectAll(".snp-cell")
        .data(vis.rows_data_slice_default);

      var visCellsEnter = visCellsUpdate
        .enter()
        .append("rect")
        .attr("class", "snp-cell");

      visCellsUpdate.exit().remove();

      visCellsEnter
        .merge(visCellsUpdate)
        // .transition()
        .attr("x", function(d, i) {
          return vis.xScaleFocus(d.pos);
        })
        .attr("y", function(d, i) {
          return vis.yScaleFocus(d.accession);
        })
        .attr("rx", 2)
        .attr("ry", 2)
        .attr("width", vis.xScaleFocus.bandwidth())
        .attr("height", vis.yScaleFocus.bandwidth())
        .style("fill", function(d) {
          return vis.colors[d.base];
        })
        .style("stroke-width", 4)
        .style("stroke", "none")
        .style("opacity", 0.8);

      vis.svgFocus
        .selectAll(".snp-cell")
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);

      // update axes
      vis.xAxisContextG.call(vis.xAxisContext);

      vis.xAxisFocusG.call(
        vis.xAxisFocus.tickValues(
          vis.xScaleFocus.domain().filter(function(d, i) {
            return !(i % 20); // tick interval
          })
        )
      );
      vis.xAxisFocusG.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line
      vis.yAxisFocusG.call(vis.yAxisFocus);
      vis.yAxisFocusG.select(".y-axis--focus .domain").remove(); // to disable rendering the axis line

      function brushed({ selection }) {
        console.log("selection brush", { selection });
        const rangeSelected = selection.map(
          vis.xScaleContext.invert,
          vis.xScaleContext
        );
        console.log(
          "range selected: ",
          // rangeSelected,
          Math.round(rangeSelected[0]),
          Math.round(rangeSelected[1])
        );

        var startUpdate = Math.round(rangeSelected[0]);
        var endUpdate = Math.round(rangeSelected[1]);
        console.log("start and end of brush: ", startUpdate, endUpdate);

        vis.xScaleFocus.domain(d3.range(startUpdate, endUpdate + 1));

        //rows data updated
        var flat_data_slice_updated = [
          vis.flat_data.pos,
          vis.flat_data.base,
          vis.flat_data.accession,
        ].map(
          vis.taker(
            vis.filter(
              vis.flat_data.pos,
              (d) => d >= startUpdate && d <= endUpdate
            )
          )
        );
        // console.log("flat data slice updated", flat_data_slice_updated);

        var flat_data_slice_updated_final = {
          pos: flat_data_slice_updated[0],
          base: flat_data_slice_updated[1],
          accession: flat_data_slice_updated[2],
        };
        // console.log("rows updated flat", flat_data_slice_updated_final);

        var length = (endUpdate - startUpdate + 1) * vis.nr_accessions;
        console.log("brush length", length);

        var rows_data_slice_updated = Array.from({ length }, (_, i) => ({
          pos: flat_data_slice_updated_final.pos[i], // position type should be changed from string to float
          base: flat_data_slice_updated_final.base[i],
          accession: flat_data_slice_updated_final.accession[i],
        }));
        // console.log("rows updated !", rows_data_slice_updated);

        vis.svgFocus.selectAll(".x-axis--focus").call(
          vis.xAxisFocus.tickValues(
            vis.xScaleFocus.domain().filter(function(d, i) {
              return !(i % 20);
            })
          )
        );

        vis.svgFocus.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line

        var visCellsUpdate = vis.svgFocus
          .selectAll(".snp-cell")
          .data(rows_data_slice_updated);

        var visCellsEnter = visCellsUpdate
          .enter()
          .append("rect")
          .attr("class", "snp-cell");

        visCellsUpdate.exit().remove();

        visCellsEnter
          .merge(visCellsUpdate)
          // .transition()
          //     .duration(800)
          .attr("x", function(d, i) {
            return vis.xScaleFocus(d.pos);
          })
          .attr("y", function(d, i) {
            return vis.yScaleFocus(d.accession);
          })
          .attr("rx", 2)
          .attr("ry", 2)
          .attr("width", vis.xScaleFocus.bandwidth())
          .attr("height", vis.yScaleFocus.bandwidth())
          .style("fill", function(d) {
            return vis.colors[d.base];
          })
          .style("stroke-width", 4)
          .style("stroke", "none")
          .style("opacity", 0.8);

        vis.svgFocus
          .selectAll(".snp-cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      // Three functions that change the tooltip when user hover / move / leave a cell
      var mouseover = function() {
        vis.tooltip.style("opacity", 0.6).style("color", "white");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "1px")
          .style("opacity", 1);
      };

      var mousemove = function(event, d) {
        vis.tooltip
          .html(
            "<strong>base:</strong> " +
              d.base +
              "<br/>" +
              "<strong>position:</strong> " +
              d.pos
          )
          // .html("base: " + d.base)
          .style("left", d3.pointer(event)[0] + 750 + "px")
          .style("top", d3.pointer(event)[1] + 150 + "px");
      };
      var mouseleave = function() {
        vis.tooltip.style("opacity", 0);
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      // function to update the MSA ordering
      function updateMSAorder(sortingOption) {
        console.log("option chosen: ", sortingOption);

        vis.yScaleFocus.domain(sortingOption);
        // vis.svgFocus
        //   .selectAll(".y-axis--focus")
        //   .transition()
        //   .duration(800)
        //   .call(vis.yAxisFocus)
        //   // .on("start", function() {
        //   //   vis.svgFocus.select(".y-axis--focus .domain").remove();
        //   // });
        vis.svgFocus.select(".y-axis--focus").call(vis.yAxisFocus);
        vis.svgFocus.select(".y-axis--focus .domain").remove(); // to disable rendering the axis line

        vis.svgFocus
          .selectAll(".snp-cell")
          .transition()
          // .duration(800)
          .attr("x", function(d, i) {
            return vis.xScaleFocus(d.pos);
          })
          .attr("y", function(d, i) {
            return vis.yScaleFocus(d.accession);
          });
        vis.svgFocus
          .selectAll(".snp-cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      // Change row ordering based on select
      d3.select("#selectButtonSort").on("change", function() {
        var selected = d3.select("#selectButtonSort").node().value;
        // console.log("selected order MSA: ", selected);

        if (selected === "alpha_asc") {
          updateMSAorder(vis.sortingOptions["alpha_asc"]);
        }
        if (selected === "alpha_desc") {
          updateMSAorder(vis.sortingOptions["alpha_desc"]);
        }
        if (selected === "phylo") {
          updateMSAorder(vis.sortingOptions["phylo"]);
        }
        if (selected === "phylo_rev") {
          updateMSAorder(vis.sortingOptions["phylo_rev"]);
        }

        if (selected === "ref_first") {
          updateMSAorder(vis.sortingOptions["ref_first"]);
        }
      });

      // Change brush size based on select
      d3.select("#selectButtonBrush").on("change", function() {
        var selectedBrush = d3.select("#selectButtonBrush").node().value;
        console.log("selected brush size: ", selectedBrush);

        var start_updated = 0;
        var end_updated = parseInt(selectedBrush) * 100;

        vis.svgContext
          .selectAll(".brush")
          .call(
            vis.brush.move,
            [start_updated, end_updated].map(vis.xScaleContext)
          );

        vis.svgFocus.selectAll(".x-axis--focus").call(
          vis.xAxisFocus.tickValues(
            vis.xScaleFocus.domain().filter(function(d, i) {
              return !(i % 20);
            })
          )
        );
        vis.svgFocus.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line
      });

      // Change brush size based on select
      d3.select("this.parentNode#selectButtonData").on("change", function() {
        var selectedData = d3.select("this.parentNode#selectButtonData").node()
          .value;
        console.log("selected data gene: ", selectedData);
      });
    },
  },
  mounted() {
    // function initVis() {
    let vis = this;

    // set dimensions
    var margin = { top: 30, right: 20, bottom: 40, left: 80 },
      width =
        d3.select("#msa_chart").node().clientWidth - margin.left - margin.right,
      height = 550 - margin.top - margin.bottom;
    var focusHeight = 30;
    vis.margin = margin;
    vis.width = width;
    vis.height = height;

    // set color scales
    var colors = {
      A: "#4daf4a",
      a: "#4daf4a",
      G: "#e41a1c",
      g: "#e41a1c",
      C: "#ff7f00",
      c: "#ff7f00",
      T: "#377eb8",
      t: "#377eb8",
      "-": "#E6E6E6",
      "*": "#686868",
    };
    vis.colors = colors;

    // legendVariants labels
    var dataLabels = [1, 2, 3, 4, 5, 6];
    var cols = [
      "#4daf4a",
      "#ff7f00",
      "#e41a1c",
      "#377eb8",
      "#E6E6E6",
      "#686868",
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
    vis.orders = orders;

    // define brush sizes
    var brushSizes = {
      1: "100 positions",
      2: "200 positions",
      3: "300 positions",
      4: "400 positions",
    };
    vis.brushSizes = brushSizes;

    // initialize scales
    var xScaleContext = d3.scaleLinear().range([0, width]);
    vis.xScaleContext = xScaleContext;

    var yScaleContext = d3.scaleLinear().range([focusHeight, 0]);
    vis.yScaleContext = yScaleContext;

    var xScaleFocus = d3
      .scaleBand()
      .range([0, width])
      .padding(0.05);
    vis.xScaleFocus = xScaleFocus;

    var yScaleFocus = d3
      .scaleBand()
      .range([height, 0])
      .padding(0.05);
    vis.yScaleFocus = yScaleFocus;

    //initialize axes
    var xAxisContext = d3.axisTop().scale(vis.xScaleContext);
    vis.xAxisContext = xAxisContext;

    var xAxisFocus = d3.axisTop(vis.xScaleFocus);
    vis.xAxisFocus = xAxisFocus;

    var yAxisFocus = d3.axisLeft(vis.yScaleFocus);
    vis.yAxisFocus = yAxisFocus;

    // initalize brush
    var brush = d3.brushX().extent([
      [0, 0],
      [width, focusHeight],
    ]);
    vis.brush = brush;

    // define svg
    var svgContext = d3
      .select("#gene_chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", focusHeight + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + 0 + ")");

    // append background
    svgContext
      .append("rect")
      .attr("class", "background-gene--context")
      .attr("width", width)
      .attr("height", focusHeight)
      .attr("transform", "translate(0," + margin.top + ")");

    // append brush
    svgContext
      .append("g")
      .attr("class", "brush")
      .attr("transform", "translate(0," + margin.top + ")");

    vis.svgContext = svgContext;

    var svgFocus = d3
      .select("#msa_chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    vis.svgFocus = svgFocus;

    // append empty axis groups
    var xAxisContextG = vis.svgContext
      .append("g")
      .attr("class", "x-axis--context")
      .attr("transform", "translate(0," + vis.margin.top + ")");
    vis.xAxisContextG = xAxisContextG;

    var xAxisFocusG = vis.svgFocus.append("g").attr("class", "x-axis--focus");
    // .style("font-size", 15)
    // .attr("transform", "translate(0," + this.height + ")")
    vis.xAxisFocusG = xAxisFocusG;

    var yAxisFocusG = vis.svgFocus.append("g").attr("class", "y-axis--focus")
    // .style("font-size", 15)
    // .select(".y-axis--focus .domain")
    // .remove();
    vis.yAxisFocusG = yAxisFocusG;

    // append legend
    var legendVariants = svgFocus
      .selectAll(".legendVariants")
      .data(dataLabels)
      .enter()
      .append("g")
      .attr("class", "legendVariants")
      .attr("transform", function(d, i) {
        return "translate(" + 50 * i + "," + 490 + ")";
      });

    legendVariants
      .append("rect")
      .attr("width", 10)
      .attr("height", 18)
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

    // add the options to the button
    d3.select("#selectButtonSort")
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
    // console.log("orders", Object.keys(orders));

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
    vis.tooltip = tooltip;

    // }
    // initVis();
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
  margin-top: 0px;
  font-weight: 700;
}

.background-gene--context {
  fill: slategray;
  fill-opacity: 0.1;
}

.variants--summary {
  stroke: black;
  stroke-width: 1.5;
  opacity: 0.2;
}

.selection {
  stroke: cornflowerblue;
  fill: cornflowerblue;
  fill-opacity: 0.3;
}
</style>
