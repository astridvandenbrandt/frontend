<template>
  <div class="selectLabelVariants">
    <!-- <label for="selectButtonGene"> Select Gene: </label>
    <select class="selectButtonVariants" id="selectButtonGene"
      ><option value="" selected disabled hidden>gene ID</option></select
    > -->
    <img
      src="./buttons/sort-down.svg"
      alt=""
      width="20"
      height="20"
      title="Sort"
    />
    <!-- <label for="selectButtonSort"> select order </label> -->
    <select class="selectButtonVariants" id="selectButtonSort"></select>
    <label for="selectButtonBrush"> Size Brush: </label>
    <select class="selectButtonVariants" id="selectButtonBrush">
      <!-- <option value="2814" selected>200 positions</option> -->
    </select>
    
    <img
    src="./buttons/sort-down.svg"
    alt=""
    width="20"
    height="20"
    title="Sort"
    id="sort_group"
  />
  <!-- <label for="selectButtonSort"> select order </label> -->
  <select class="selectButtonVariants" id="selectButtonSortPhenos"></select>
  </div>
  <div class="container">
    <div id="chart"></div>
    <div class="row" id="header-arrows">
      <div class="col" id="footer-col-right">
        <div class="float-start border rounded" id="btn-arrow-left">
          <button type="button" class="btn btn-link btn-sm" id="arrow-left">
            <img
              src="./buttons/arrow-left.svg"
              alt=""
              width="12"
              height="12"
              title="arrow-left"
            />
          </button>
        </div>
      </div>
      <div class="col" id="footer-col-left">
        <div class="float-end border rounded" id="btn-arrow-right">
          <button type="button" class="btn btn-link btn-sm" id="arrow-right">
            <img
              src="./buttons/arrow-right.svg"
              alt=""
              width="12"
              height="12"
              title="arrow-right"
            />
          </button>
        </div>
      </div>
    </div>
  </div>

  <div id="tooltip"></div>
  <div id="tooltip-bars"></div>
  <div id="tooltip-bars--pheno"></div>
  <div id="tooltip-pheno"></div>
  <div id="tooltip-bars--pheno2"></div>
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

      // if (vis.globalBrushStart == undefined){
      //   console.log(" global brush values undefined")
      // }

      console.log('global brush start value from UpdateVis', vis.globalBrushStart)
      console.log('global brush start value from UpdateVis', vis.globalBrushEnd)


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

      const mutations = data_mutations.map((d) => parseFloat(d.accession));
      const max_mutations = d3.max(mutations);
      vis.max_mutations = max_mutations;
      console.log("max mutations", mutations, max_mutations);


      // else {
        var start = vis.globalBrushStart;

        var end = vis.globalBrushEnd

      // }
      if (vis.globalBrushStart > length_gene || vis.globalBrushEnd > length_gene){
        console.log("start exceeds length new gene!!")
        // start = length_gene - 50
        // end = length_gene
        start = 0
        end = 50

      }

      // if (vis.globalBrushEnd > length_gene){
      //   console.log("end exceeds length new gene!!")
      //   // // start = length_gene - 50
      //   end = length_gene

      // }

      vis.start = start;
      vis.end = end;
 
      const updateBrush = d3.select("#selectButtonBrush").node().value;
      console.log('update brush with new data', updateBrush)
      // const end = updateBrush * 100; // when new data loaded keep brush size
    
      const updateSort = d3.select("#selectButtonSort").node().value; // when new data loaded keep sorting order

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
      // console.log("genePositions", genePositions);

      // var alpha = [
      //   "8__Tsu-0",
      //   "8__Kas-1",
      //   "8__Altai-5",
      //   "8_Sha",
      //   "7__Sku-30",
      //   "7__Ler-0",
      //   "7__Gro-3",
      //   "7_Ler",
      //   "6__Tsu-0",
      //   "6__Kas-1",
      //   "6__Altai-5",
      //   "6_Kyo",
      //   "5__Sku-30",
      //   "5__Ler-0",
      //   "5__Gro-3",
      //   "5_Eri",
      //   "4_Cvi",
      //   "3_C24",
      //   "2_An-1",
      //   "1__Tsu-0",
      //   "1__Sku-30",
      //   "1__Ler-0",
      //   "1__Kas-1",
      //   "1__Gro-3",
      //   "1__Altai-5",
      //   "1_Col-0",
      // ];
      // var alpha = [
      //   "8_Sha",
      //   "7_Ler",
      //   "6_Kyo",
      //   "5_Eri",
      //   "4_Cvi",
      //   "3_C24",
      //   "2_An-1",
      //   "Tsu-0",
      //   "Sku-30",
      //   "Ler-0",
      //   "Kas-1",
      //   "Gro-3",
      //   "Altai-5",
      //   "1_Col-0",
      // ];

      var phylo_kmer = [
        "Tsu-0",
        "Sku-30",
        "Ler-0",
        "Kas-1",
        "Gro-3",
        "Altai-5",
        "2_An-1",
        "1_Col-0",
        "4_Cvi",
        "3_C24",
        "8_Sha",
        "7_Ler",
        "5_Eri",
        "6_Kyo",
      ];
      // var phylo_kmer = [
      //   "2_An-1",
      //   "1__Tsu-0",
      //   "1__Sku-30",
      //   "1__Ler-0",
      //   "1__Kas-1",
      //   "1__Gro-3",
      //   "1__Altai-5",
      //   "1_Col-0",
      //   "4_Cvi",
      //   "3_C24",
      //   "8__Tsu-0",
      //   "8__Kas-1",
      //   "8__Altai-5",
      //   "8_Sha",
      //   "7__Sku-30",
      //   "7__Ler-0",
      //   "7__Gro-3",
      //   "7_Ler",
      //   "5__Sku-30",
      //   "5__Ler-0",
      //   "5__Gro-3",
      //   "5_Eri",
      //   "6__Tsu-0",
      //   "6__Kas-1",
      //   "6__Altai-5",
      //   "6_Kyo",
      // ];

      var ref_first = [
        "Gro-3",
        "Ler-0",
        "Sku-30",
        "Altai-5",
        "Kas-1",
        "Tsu-0",
        "8_Sha",
        "7_Ler",
        "6_Kyo",
        "5_Eri",
        "4_Cvi",
        "3_C24",
        "2_An-1",
        "1_Col-0",
      ];

      var original_map = [
        "8_Sha",
        "7_Ler",
        "6_Kyo",
        "5_Eri",
        "4_Cvi",
        "3_C24",
        "2_An-1",
        "Gro-3",
        "Ler-0",
        "Sku-30",
        "Altai-5",
        "Kas-1",
        "Tsu-0",
        "1_Col-0",
      ];

      var group = [
        "1_Col-0",
        "2_An-1",
        "4_Cvi",
        "Altai-5",
        "Sku-30",
        "Ler-0",
        "3_C24",
        "5_Eri",
        "6_Kyo",
        "7_Ler",
        "8_Sha",
        "Tsu-0",
        "Kas-1",
        "Gro-3",
      ];

      var origin = [
        "1_Col-0",
        "8_Sha",
        "Sku-30",
        "Gro-3",
        "5_Eri",
        "3_C24",
        "Tsu-0",
        "6_Kyo",
        "Kas-1",
        "Ler-0",
        "7_Ler",
        "Altai-5",
        "4_Cvi",
        "2_An-1",
      ];

      var dtf3 = [
        "Tsu-0",
        "1_Col-0",
        "Ler-0",
        "7_Ler", 
        "3_C24",
        "5_Eri",
        "6_Kyo",
        "2_An-1",
        "4_Cvi",
        "8_Sha",
        "Altai-5",
        "Gro-3",
        "Kas-1",
        "Sku-30",
      ];

      var dtf1 = [
        "Tsu-0",
        "1_Col-0",
        "7_Ler", 
        "Ler-0",
        "3_C24",
        "6_Kyo",
        "2_An-1",
        "4_Cvi",
        "Kas-1",
        "8_Sha",
        "Altai-5",
        "5_Eri",
        "Sku-30",
        "Gro-3",
      ];

      // var ref_first = [
      //   "8__Tsu-0",
      //   "8__Altai-5",
      //   "8__Kas-1",
      //   "5__Gro-3",
      //   "5__Ler-0",
      //   "5__Sku-30",
      //   "6__Altai-5",
      //   "6__Kas-1",
      //   "6__Tsu-0",
      //   "7__Sku-30",
      //   "7__Gro-3",
      //   "7__Ler-0",
      //   "1__Gro-3",
      //   "1__Ler-0",
      //   "1__Sku-30",
      //   "1__Altai-5",
      //   "1__Kas-1",
      //   "1__Tsu-0",
      //   "2_An-1",
      //   "8_Sha",
      //   "5_Eri",
      //   "6_Kyo",
      //   "3_C24",
      //   "7_Ler",
      //   "4_Cvi",
      //   "1_Col-0",
      // ];

      var sortingOptions = {
        original_map: original_map,
        dtf1: dtf1,
        dtf1_rev: dtf1.slice().reverse(),
        dtf3: dtf3,
        dtf3_rev: dtf3.slice().reverse(),
        group: group,
        group_rev: group.slice().reverse(),
        origin: origin,
        origin_rev: origin.slice().reverse(),
        ref_first: ref_first,
        ref_last: ref_first.slice().reverse(),
        // alpha_asc: alpha,
        // alpha_desc: alpha.slice().reverse(),
        phylo: phylo_kmer,
        phylo_rev: phylo_kmer.slice().reverse(),
      };
      vis.sortingOptions = sortingOptions;

      var sortingOptionsPheno = {
        original_map: original_map,
        dtf1: dtf1,
        dtf1_rev: dtf1.slice().reverse(),
        dtf3: dtf3,
        dtf3_rev: dtf3.slice().reverse(),
        group: group,
        group_rev: group.slice().reverse(),
        origin: origin,
        origin_rev: origin.slice().reverse(),
        ref_first: ref_first,
        ref_last: ref_first.slice().reverse(),
        phylo: phylo_kmer,
        phylo_rev: phylo_kmer.slice().reverse(),
      };
      vis.sortingOptionsPheno = sortingOptionsPheno;

      // Set the scale input domains
      vis.xScaleContext.domain([0, vis.length_gene]);
      vis.yScaleContext.domain([0, vis.max_mutations]); // 11 is max vars on one pos
      vis.xScaleFocus.domain(genePositions);
      vis.yScaleFocus.domain(sortingOptions[updateSort]);
      vis.xScalePhenos.domain(['Group','Origin','DTF1','DTF3',]);
      vis.yScalePhenos.domain(sortingOptions[updateSort]);
      vis.xScalePhenoBars.domain([0, 115.33]);
      vis.yScalePhenoBars.domain(sortingOptions[updateSort]);
      vis.xScalePhenoBars2.domain([0, 137]);
      vis.yScalePhenoBars2.domain(sortingOptions[updateSort]);

      var length = (end - start + 1) * nr_accessions; //nr of cells to render

      var rows_data_slice_default = Array.from({ length }, (_, i) => ({
        pos: flat_data_slice_default.pos[i],
        base: flat_data_slice_default.base[i],
        accession: flat_data_slice_default.accession[i],
      }));
      console.log("rows_data_slice_default", rows_data_slice_default);
      vis.rows_data_slice_default = rows_data_slice_default;

      var data_pheno_bars = [
      {"accession": "8_Sha", "pheno": "DTF3", "value":58.5},
        {"accession": "7_Ler", "pheno": "DTF3", "value": 40},
        {"accession": "6_Kyo", "pheno": "DTF3", "value": 49.75},
        {"accession": "5_Eri", "pheno": "DTF3", "value": 42.75},
        {"accession": "4_Cvi",  "pheno": "DTF3", "value": 56},
        {"accession": "3_C24", "pheno": "DTF3", "value": 42.5},
        {"accession": "2_An-1", "pheno": "DTF3", "value": 54.25},
        {"accession": "Tsu-0", "pheno": "DTF3", "value": 28},
        {"accession": "Sku-30", "pheno": "DTF3", "value": 115.33},
        {"accession": "Ler-0",  "pheno": "DTF3", "value": 40},
        {"accession": "Kas-1",  "pheno": "DTF3", "value":80},
        {"accession": "Gro-3",  "pheno": "DTF3", "value":78},
        {"accession": "Altai-5",  "pheno": "DTF3", "value":73},
        {"accession": "1_Col-0",  "pheno": "DTF3", "value":38},
      ];

      vis.data_pheno_bars = data_pheno_bars;


      var data_pheno_bars_2 = [

        {"accession": "8_Sha", "pheno": "DTF1", "value":72},
        {"accession": "7_Ler", "pheno": "DTF1", "value": 28.5},
        {"accession": "6_Kyo", "pheno": "DTF1", "value": 40},
        {"accession": "5_Eri", "pheno": "DTF1", "value": 85},
        {"accession": "4_Cvi",  "pheno": "DTF1", "value": 61},
        {"accession": "3_C24", "pheno": "DTF1", "value": 32.667},
        {"accession": "2_An-1", "pheno": "DTF1", "value": 42},
        {"accession": "Tsu-0", "pheno": "DTF1", "value": 28},
        {"accession": "Sku-30", "pheno": "DTF1", "value": 102.33},
        {"accession": "Ler-0",  "pheno": "DTF1", "value": 29},
        {"accession": "Kas-1",  "pheno": "DTF1", "value":69},
        {"accession": "Gro-3",  "pheno": "DTF1", "value":137},
        {"accession": "Altai-5",  "pheno": "DTF1", "value":73},
        {"accession": "1_Col-0",  "pheno": "DTF1", "value":28},

      ]
      vis.data_pheno_bars_2 = data_pheno_bars_2;

      var data_pheno = [
        {"accession": "8_Sha", "pheno": "DTF1", "value":72},
        {"accession": "7_Ler", "pheno": "DTF1", "value": 28.5},
        {"accession": "6_Kyo", "pheno": "DTF1", "value": 40},
        {"accession": "5_Eri", "pheno": "DTF1", "value": 85},
        {"accession": "4_Cvi",  "pheno": "DTF1", "value": 61},
        {"accession": "3_C24", "pheno": "DTF1", "value": 32.667},
        {"accession": "2_An-1", "pheno": "DTF1", "value": 42},
        {"accession": "Tsu-0", "pheno": "DTF1", "value": 28},
        {"accession": "Sku-30", "pheno": "DTF1", "value": 102.33},
        {"accession": "Ler-0",  "pheno": "DTF1", "value": 29},
        {"accession": "Kas-1",  "pheno": "DTF1", "value":69},
        {"accession": "Gro-3",  "pheno": "DTF1", "value":137},
        {"accession": "Altai-5",  "pheno": "DTF1", "value":73},
        {"accession": "1_Col-0",  "pheno": "DTF1", "value":28},

        {"accession": "8_Sha", "pheno": "DTF3", "value":58.5},
        {"accession": "7_Ler", "pheno": "DTF3", "value": 40},
        {"accession": "6_Kyo", "pheno": "DTF3", "value": 49.75},
        {"accession": "5_Eri", "pheno": "DTF3", "value": 42.75},
        {"accession": "4_Cvi",  "pheno": "DTF3", "value": 56},
        {"accession": "3_C24", "pheno": "DTF3", "value": 42.5},
        {"accession": "2_An-1", "pheno": "DTF3", "value": 54.25},
        {"accession": "Tsu-0", "pheno": "DTF3", "value": 28},
        {"accession": "Sku-30", "pheno": "DTF3", "value": 115.33},
        {"accession": "Ler-0",  "pheno": "DTF3", "value": 40},
        {"accession": "Kas-1",  "pheno": "DTF3", "value":80},
        {"accession": "Gro-3",  "pheno": "DTF3", "value":78},
        {"accession": "Altai-5",  "pheno": "DTF3", "value":73},
        {"accession": "1_Col-0",  "pheno": "DTF3", "value":38},

        {"accession": "8_Sha", "pheno": "Group", "value":'X'},
        {"accession": "7_Ler", "pheno": "Group", "value": 'X'},
        {"accession": "6_Kyo", "pheno": "Group", "value": 'X'},
        {"accession": "5_Eri", "pheno": "Group", "value": 'X'},
        {"accession": "4_Cvi",  "pheno": "Group", "value": 'Y'},
        {"accession": "3_C24", "pheno": "Group", "value": 'X'},
        {"accession": "2_An-1", "pheno": "Group", "value": 'Y'},
        {"accession": "Tsu-0", "pheno": "Group", "value": 'X'},
        {"accession": "Sku-30", "pheno": "Group", "value": 'Y'},
        {"accession": "Ler-0",  "pheno": "Group", "value": 'Y'},
        {"accession": "Kas-1",  "pheno": "Group", "value":'X'},
        {"accession": "Gro-3",  "pheno": "Group", "value":'X'},
        {"accession": "Altai-5",  "pheno": "Group", "value":'Y'},
        {"accession": "1_Col-0",  "pheno": "Group", "value":'Y'},


        {"accession": "8_Sha", "pheno": "Origin", "value":'Tajikistan'},
        {"accession": "7_Ler", "pheno": "Origin", "value": 'Germany'},
        {"accession": "6_Kyo", "pheno": "Origin", "value": 'Japan'},
        {"accession": "5_Eri", "pheno": "Origin", "value": 'Sweden'},
        {"accession": "4_Cvi",  "pheno": "Origin", "value": 'Cape Verde'},
        {"accession": "3_C24", "pheno": "Origin", "value": 'Portugal'},
        {"accession": "2_An-1", "pheno": "Origin", "value": 'Belgium'},
        {"accession": "Tsu-0", "pheno": "Origin", "value": 'Japan'},
        {"accession": "Sku-30", "pheno": "Origin", "value": 'Sweden'},
        {"accession": "Ler-0",  "pheno": "Origin", "value": 'Germany'},
        {"accession": "Kas-1",  "pheno": "Origin", "value":'India'},
        {"accession": "Gro-3",  "pheno": "Origin", "value":'Sweden'},
        {"accession": "Altai-5",  "pheno": "Origin", "value":'China'},
        {"accession": "1_Col-0",  "pheno": "Origin", "value":'United States'}
      ]


      console.log("data phenos", data_pheno)

      vis.data_pheno = data_pheno;

      vis.renderVis();
    },
    // This function contains the D3 code for binding data to visual elements.
    renderVis() {
      let vis = this;

      // PHENO BARS 2
      console.log("data pheno bars 2:", vis.data_pheno_bars_2);
      // check new data
      let visPhenoBarUpdate2 = vis.phenoBars2
        .selectAll(".pheno-bar2")
        .data(vis.data_pheno_bars_2);

      // make new rects
      let visPhenoBarEnter2 = visPhenoBarUpdate2
        .enter()
        .append("rect")
        .attr("class", "pheno-bar2");

      // remove old rects
      visPhenoBarUpdate2.exit().remove();

      //merge rects
      visPhenoBarEnter2
        .merge(visPhenoBarUpdate2)
        .attr("width", (d) => vis.xScalePhenoBars2(d.value))
        .attr("height", vis.yScalePhenoBars2.bandwidth())
        .attr("x", 0)
        .attr("y", (d) => vis.yScalePhenoBars2(d.accession))
        .attr('fill', '#2a8d46')
        .style("opacity", 0.8);


      // PHENO BARS 
      console.log("data pheno bars:", vis.data_pheno_bars);
      // check new data
      let visPhenoBarUpdate = vis.phenoBars
        .selectAll(".pheno-bar")
        .data(vis.data_pheno_bars);

      // make new rects
      let visPhenoBarEnter = visPhenoBarUpdate
        .enter()
        .append("rect")
        .attr("class", "pheno-bar");

      // remove old rects
      visPhenoBarUpdate.exit().remove();

      //merge rects
      visPhenoBarEnter
        .merge(visPhenoBarUpdate)
        .attr("width", (d) => vis.xScalePhenoBars(d.value))
        .attr("height", vis.yScalePhenoBars.bandwidth())
        .attr("x", 0)
        .attr("y", (d) => vis.yScalePhenoBars(d.accession))
        .attr('fill', 'steelblue')
        .style("opacity", 0.8);


      console.log("data mutations new:", vis.data_mutations);
      // check new data
      let visGeneUpdate = vis.svgContext
        .selectAll(".variants--summary-bar")
        .data(vis.data_mutations);

      // make new rects
      let visGeneEnter = visGeneUpdate
        .enter()
        .append("rect")
        .attr("class", "variants--summary-bar");

      // remove old rects
      visGeneUpdate.exit().remove();

      //merge rects
      visGeneEnter
        .merge(visGeneUpdate)
        .attr("height", (d) => vis.focusHeight - vis.yScaleContext(d.accession))
        .attr("width", 1.5)
        .attr("x", (d) => vis.xScaleContext(d.pos))
        .attr("y", (d) => vis.yScaleContext(d.accession));

      vis.svgContext
        .selectAll(".brush")
        .call(vis.brush)
        .call(vis.brush.move, [vis.start, vis.end].map(vis.xScaleContext));

      console.log(
        "test brush inital values",
        vis.start,
        [vis.start, vis.end].map(vis.xScaleContext)[1]
      );

      // console.log('test brushed labels', brushedReload)

      // update brush labels
      updateLabelBrush("left", vis.start, vis.end);
      updateLabelBrush("right", vis.start, vis.end);

      // update figure when brushing
      vis.brush.on("end", brushed); //change 'end' to 'brush' if want to see inbetween
      // use brush for labels!!
      vis.brush.on("brush", brushUpdate);

      updateVariantFocusChart(vis.rows_data_slice_default);

      // check new data
      let visPhenoUpdate = vis.phenoChart
          .selectAll(".pheno-cell")
          .data(vis.data_pheno, (d) => d); //key function?

        // make new cells
        let visPhenoEnter = visPhenoUpdate
          .enter()
          .append("rect")
          .attr("class", "pheno-cell");

        // remove old cells
        visPhenoUpdate.exit().remove();


        // merge cells with existing
        visPhenoEnter
          .merge(visPhenoUpdate)
          // .transition()
          //     .duration(300)
          .attr("x", (d) => vis.xScalePhenos(d.pheno))
          .attr("y", (d) => vis.yScalePhenos(d.accession))
          .attr("rx", 2.5)
          .attr("ry", 2.5)
          .attr("width", vis.xScalePhenos.bandwidth())
          .attr("height", vis.yScalePhenos.bandwidth())
          .style("fill", function(d) {
            if (d.pheno == "DTF1"){
              return vis.colorScaleDTF1(d.value)
            }
            if (d.pheno == "DTF3"){
              return vis.colorScaleDTF3(d.value)
            }
            if (d.pheno == "Group"){
              return vis.colorScaleGroup(d.value)
            }
            if (d.pheno == "Origin"){
              return vis.colorScaleOrigin(d.value)
            }
          
          })
          .style("stroke-width", 1)
          .style("stroke", "dimgrey")
          .style("opacity", 0.9);

      
      // update axes
      vis.xAxisContextG.call(vis.xAxisContext);
      vis.yAxisContextG.call(vis.yAxisContext);

      vis.xAxisFocusG.call(
        vis.xAxisFocus.tickValues(
          vis.xScaleFocus.domain().filter(function(d, i) {
            return !(i % 10); // defines tick interval
          })
        )
      );
      vis.xAxisFocusG.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line
      vis.yAxisFocusG.call(vis.yAxisFocus);
      vis.yAxisFocusG.select(".y-axis--focus .domain").remove(); // to disable rendering the axis line
      
      vis.xAxisPhenosG.call(vis.xAxisPhenos);
      vis.xAxisPhenosG.select(".x-axis--phenos .domain").remove(); // to disable rendering the axis line
      vis.yAxisPhenosG.call(vis.yAxisPhenos);
      vis.yAxisPhenosG.select(".y-axis--phenos .domain").remove(); // to disable rendering the axis line

      vis.xAxisPhenoBarsG.call(vis.xAxisPhenoBars);
      vis.yAxisPhenoBarsG.call(vis.yAxisPhenoBars);
      vis.yAxisPhenoBarsG.select(".y-axis--phenobars .domain").remove(); // to disable rendering the axis line


      vis.xAxisPhenoBarsG2.call(vis.xAxisPhenoBars2);
      vis.yAxisPhenoBarsG2.call(vis.yAxisPhenoBars2);
      vis.yAxisPhenoBarsG2.select(".y-axis--phenobars2 .domain").remove(); // to disable rendering the axis line

      
      function brushValues({selection}) {
        


        const rangeSelected = selection.map(
          vis.xScaleContext.invert,
          vis.xScaleContext
        );
        console.log(
          "range selected brush: ",
          // rangeSelected,
          Math.round(rangeSelected[0]),
          Math.round(rangeSelected[1])
        );

        var startUpdate = Math.round(rangeSelected[0]);
        var endUpdate = Math.round(rangeSelected[1]);

        return [startUpdate, endUpdate]



      }

      function brushUpdate({ selection }) {
        console.log("selection brush end", { selection });
        const rangeSelected = selection.map(
          vis.xScaleContext.invert,
          vis.xScaleContext
        );
        console.log(
          "range selected brush end: ",
          // rangeSelected,
          Math.round(rangeSelected[0]),
          Math.round(rangeSelected[1])
        );

        var startUpdate = Math.round(rangeSelected[0]);
        var endUpdate = Math.round(rangeSelected[1]);

        updateLabelBrush("left", startUpdate, endUpdate);
        updateLabelBrush("right", startUpdate, endUpdate);

        vis.globalBrushStart = startUpdate
        vis.globalBrushEnd = endUpdate;
        console.log('global brush values', vis.globalBrushStart, vis.globalBrushEnd)
      }

      // updates labels brush
      function updateLabelBrush(side, start, end) {
        if (side == "left") {
          var brushClass = ".brushLabelL";
          var style = "end";
          var data = [start];
          var xPos = 0;
        } else {
          brushClass = ".brushLabelR";
          style = "start";
          data = [end];
          xPos = 1;
        }
        const textUpdate = vis.svgContextLabels
          .selectAll(brushClass)
          .data(data);

        textUpdate.selectAll("text").remove(); //remove old label

        const textEnter = textUpdate.append("text");

        const textExit = textUpdate.exit().remove();

        textEnter
          .merge(textUpdate)
          .style("text-anchor", style)
          .attr("x", [start, end].map(vis.xScaleContext)[xPos])
          .text((d) => d);
      }

      function updateVariantFocusChart(data) {
        var row = vis.svgFocus
          .selectAll(".row--accession")
          .data([...new Set(data.map((d) => d.accession))])
          .enter()
          .append("svg:g")
          .attr("class", ".row--accession");

        //  // check new data
        //  let visCellsUpdate = row
        //   .selectAll(".snp-cell")
        //   .data(function (d,i) {
        //     console.log(' data for cells matrix: ', d.map(function(a) { return {value: a, row: i}; } ))
        //     return d.map(function(a) { return {value: a, row: i}; } ) })

        // check new data
        let visCellsUpdate = vis.svgFocus
          .selectAll(".snp-cell")
          .data(data, (d) => d); //key function?

        // make new cells
        let visCellsEnter = visCellsUpdate
          .enter()
          .append("rect")
          .attr("class", "snp-cell");

        // remove old cells
        visCellsUpdate.exit().remove();

        // merge cells with existing
        visCellsEnter
          .merge(visCellsUpdate)
          // .transition()
          //     .duration(300)
          .attr("x", (d) => vis.xScaleFocus(d.pos))
          .attr("y", (d) => vis.yScaleFocus(d.accession))
          .attr("rx", 1.5)
          .attr("ry", 1.5)
          .attr("width", vis.xScaleFocus.bandwidth())
          .attr("height", vis.yScaleFocus.bandwidth())
          .style("fill", (d) => vis.colors[d.base])
          // .style("stroke-width", 0.5)
          // .style("stroke", "white")
          .style("opacity", 0.8);
      }

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

        // console.log("brush range", d3.extent(vis.xScaleFocus.domain()));

        // // update brush labels
        // updateLabelBrush("left",startUpdate,endUpdate);
        // updateLabelBrush("right",startUpdate,endUpdate);

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
        console.log("nr of cells to render", length);

        var rows_data_slice_updated = Array.from({ length }, (_, i) => ({
          pos: flat_data_slice_updated_final.pos[i], // position type should be changed from string to float
          base: flat_data_slice_updated_final.base[i],
          accession: flat_data_slice_updated_final.accession[i],
        }));
        // console.log("rows updated !", rows_data_slice_updated);

        vis.svgFocus.selectAll(".x-axis--focus").call(
          vis.xAxisFocus.tickValues(
            vis.xScaleFocus.domain().filter(function(d, i) {
              return !(i % 20); // defines tick interval
            })
          )
        );
        vis.svgFocus.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line

        updateVariantFocusChart(rows_data_slice_updated);

        vis.svgContext
          .selectAll(".variants--summary-bar")
          .on("mouseover", mouseoverBar)
          .on("mousemove", mousemoveBar)
          .on("mouseleave", mouseleaveBar);

        vis.svgFocus
          .selectAll(".snp-cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      var mouseover = function() {
        d3.select("#tooltip")
          .style("display", "block")
          .style("opacity", 0.8)
          // .style("color", "white");
          .style("color", "black");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "1px")
          .style("opacity", 1);
      };


      var mousemove = function(event, d) {
        d3.select("#tooltip")
          // .style("display", "block")
          .html(
            "<strong>base:</strong> " +
              d.base +
              "<br/>" +
              "<strong>position:</strong> " +
              d.pos +
              "<br/>" +
              "<strong>accession:</strong> " +
              d.accession
          )
          // .style("left", d3.pointer(event)[0] + 530 + "px")
          // .style("top", d3.pointer(event)[1] + 150 + "px");

          .style("left", d3.pointer(event)[0] + (vis.leftColWidth*2) + "px")
          .style("top", d3.pointer(event)[1]  + (vis.topRowHeight*2) + "px");

     
      };

      var mouseleave = function() {
        d3.select("#tooltip").style("display", "none");
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      var mouseleaveBar = function() {
        d3.select("#tooltip-bars").style("display", "none");
        d3.select(this)
          .style("stroke", "none")
          .style("opacity", 0.8);
      };

      var mouseoverBar = function() {
        d3.select("#tooltip-bars")
          .style("display", "block")
          .style("opacity", 0.8)
          // .style("color", "white");
          .style("color", "black");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "1px")
          .style("opacity", 1);
      };

      var mousemoveBar = function(event, d) {
        d3.select("#tooltip-bars")
          // .style("display", "block")
          .html(
            "<strong>#SNPs:</strong> " +
              d.accession +
              "<br/>" +
              "<strong>Position:</strong> " +
              d.pos
          )
          .style("left", d3.pointer(event)[0] + (vis.leftColWidth*2) + "px")
          .style("top", d3.pointer(event)[1] + (vis.topRowHeight) + "px");
      };

      var mouseleavePheno = function() {
        d3.select("#tooltip-pheno").style("display", "none");
        d3.select(this)
         .style("stroke", "darkgrey")
          .style("stroke-width", "1px")
          .style("opacity", 0.8);
      };

      var mouseoverPheno = function() {
        d3.select("#tooltip-pheno")
          .style("display", "block")
          .style("opacity", 0.8)
          .style("color", "white");
          // .style("color", "black");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "2px")
          .style("opacity", 1);
      };

      var mousemovePheno = function(event, d) {
        d3.select("#tooltip-pheno")
          // .style("display", "block")
          .html(
            d.accession +
              "<br/>" +
              d.value
          )
          .style("left", d3.pointer(event)[0] + (vis.leftColWidth*2.2 + vis.midColWidth) + "px")
          .style("top", d3.pointer(event)[1] + (vis.topRowHeight*7) + "px");
      };

      var mouseleavePhenoBar = function() {
        d3.select("#tooltip-bars--pheno").style("display", "none");
        d3.select(this)
          .style("stroke", "white")
          .style("stroke-width", "1px")
          .style("opacity", 0.8);
      };

      var mouseoverPhenoBar = function() {
        d3.select("#tooltip-bars--pheno")
          .style("display", "block")
          .style("opacity", 0.8)
          .style("color", "white");
          // .style("color", "black");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "2px")
          .style("opacity", 1);
     
      };

      var mousemovePhenoBar = function(event, d) {
        d3.select("#tooltip-bars--pheno")
          // .style("display", "block")
          .html(
              d.accession
              + "<br/>" +
              d.value

          )
          .style("left", d3.pointer(event)[0] + (vis.leftColWidth*2 + vis.midColWidth + vis.phenoChartWidth) + "px")
          .style("top", d3.pointer(event)[1] + (vis.topRowHeight*7) + "px");
      };

      var mouseleavePhenoBar2 = function() {
        d3.select("#tooltip-bars--pheno2").style("display", "none");
        d3.select(this)
          .style("stroke", "white")
          .style("stroke-width", "1px")
          .style("opacity", 0.8);
      };

      var mouseoverPhenoBar2 = function() {
        d3.select("#tooltip-bars--pheno2")
          .style("display", "block")
          .style("opacity", 0.8)
          .style("color", "white");
          // .style("color", "black");
        d3.select(this)
          .style("stroke", "black")
          .style("stroke-width", "2px")
          .style("opacity", 1);
     
      };

      var mousemovePhenoBar2 = function(event, d) {
        d3.select("#tooltip-bars--pheno2")
          // .style("display", "block")
          .html(
            d.accession
              + "<br/>" +
              d.value
          )
          .style("left", d3.pointer(event)[0] + (vis.leftColWidth*2 + vis.midColWidth + vis.phenoChartWidth*0.6) + "px")
          .style("top", d3.pointer(event)[1] + (vis.topRowHeight*7) + "px");
      };

      // function to update the MSA ordering
      function updateMSAorder(sortingOption) {
        console.log("option chosen: ", sortingOption);

        vis.yScaleFocus.domain(sortingOption);
        vis.svgFocus.select(".y-axis--focus").call(vis.yAxisFocus);
        vis.svgFocus.select(".y-axis--focus .domain").remove(); // to disable rendering the axis line

        vis.svgFocus
          .selectAll(".snp-cell")
          .transition()
          .duration(500)
          .attr("x", (d) => vis.xScaleFocus(d.pos))
          .attr("y", (d) => vis.yScaleFocus(d.accession));
        vis.svgFocus
          .selectAll(".snp-cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      // Change row ordering based on select
      d3.select("#selectButtonSort").on("change", function() {
        var selected = d3.select("#selectButtonSort").node().value;
        console.log("selected order MSA: ", selected);
        updateMSAorder(vis.sortingOptions[selected]);
        d3.select('#selectButtonSortPhenos').property('value', selected);
        updatePhenosOrder(vis.sortingOptionsPheno[selected])
      });

      // function to update the MSA ordering
      function updatePhenosOrder(sortingOption) {
        console.log("option chosen: ", sortingOption);

        vis.yScalePhenos.domain(sortingOption);
        vis.phenoChart.select(".y-axis--phenos").call(vis.yAxisPhenos);
        vis.phenoChart.select(".y-axis--phenos .domain").remove(); // to disable rendering the axis line

        vis.phenoChart
          .selectAll(".pheno-cell")
          .transition()
          .duration(500)
          .attr("x", (d) => vis.xScalePhenos(d.pheno))
          .attr("y", (d) => vis.yScalePhenos(d.accession))


        vis.yScalePhenoBars.domain(sortingOption);
        vis.phenoBars.select(".y-axis--phenobars").call(vis.yAxisPhenoBars);
        vis.phenoBars.select(".y-axis--phenobars .domain").remove(); // to disable rendering the axis line

        vis.phenoBars
          .selectAll(".pheno-bar")
          .transition()
          .duration(500)
          .attr("width", (d) => vis.xScalePhenoBars(d.value))
          .attr("height", vis.yScalePhenoBars.bandwidth())
          .attr("x", 0)
          .attr("y", (d) => vis.yScalePhenoBars(d.accession))


        vis.yScalePhenoBars2.domain(sortingOption);
        vis.phenoBars2.select(".y-axis--phenobars2").call(vis.yAxisPhenoBars);
        vis.phenoBars2.select(".y-axis--phenobars2 .domain").remove(); // to disable rendering the axis line

        vis.phenoBars2
          .selectAll(".pheno-bar2")
          .transition()
          .duration(500)
          .attr("width", (d) => vis.xScalePhenoBars2(d.value))
          .attr("height", vis.yScalePhenoBars2.bandwidth())
          .attr("x", 0)
          .attr("y", (d) => vis.yScalePhenoBars2(d.accession))

        vis.phenoChart
        .selectAll(".pheno-cell")
          .on("mouseover", mouseoverPheno)
          .on("mousemove", mousemovePheno)
          .on("mouseleave", mouseleavePheno);


        vis.phenoBars
        .selectAll(".pheno-bar")
          .on("mouseover", mouseoverPhenoBar)
          .on("mousemove", mousemovePhenoBar)
          .on("mouseleave", mouseleavePhenoBar);

          vis.phenoBars2
        .selectAll(".pheno-bar")
          .on("mouseover", mouseoverPhenoBar2)
          .on("mousemove", mousemovePhenoBar2)
          .on("mouseleave", mouseleavePhenoBar2);
          
       
      }

      // Change row ordering based on select
      d3.select("#selectButtonSortPhenos").on("change", function() {
        var selected = d3.select("#selectButtonSortPhenos").node().value;
        console.log("selected order phenos: ", selected);
        updatePhenosOrder(vis.sortingOptionsPheno[selected]);
        d3.select('#selectButtonSort').property('value', selected);
        updateMSAorder(vis.sortingOptions[selected])
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

        console.log(
          "test brush values",
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

      // Change with arrow left
      d3.select("#btn-arrow-left").on("click", function() {
        console.log("clicked left arrow");
        brushArrow("left");
      });

      // Change focus with arrow right
      d3.select("#btn-arrow-right").on("click", function() {
        console.log("clicked right arrow");
        brushArrow("right");
      });

      function brushArrow(arrow) {
        const rangeOld = d3.extent(vis.xScaleFocus.domain());
        console.log("old brush start", rangeOld[0]);
        console.log("old brush end", rangeOld[1]);

        if (arrow === "left") {
          if (rangeOld[0] <= 0) {
            var startUpdate = 0;
            var endUpdate = rangeOld[1] - 1;
          } else {
            startUpdate = rangeOld[0] - 1;
            endUpdate = rangeOld[1] - 1;
            console.log(
              "new start and end of left brush: ",
              startUpdate,
              endUpdate
            );
          }
        }
        if (arrow == "right") {
          if (rangeOld[1] >= vis.length_gene) {
            startUpdate = rangeOld[0] + 1;
            endUpdate = vis.length_gene;
          } else {
            startUpdate = rangeOld[0] + 1;
            endUpdate = rangeOld[1] + 1;
            console.log(
              "new start and end of right brush: ",
              startUpdate,
              endUpdate
            );
          }
        }

        // update brush selection
        vis.svgContext
          .selectAll(".brush")
          .call(
            vis.brush.move,
            [startUpdate, endUpdate].map(vis.xScaleContext)
          );

        // update focus domain
        vis.xScaleFocus.domain(d3.range(startUpdate, endUpdate + 1));
        // console.log('brush range', d3.extent(vis.xScaleFocus.domain()))

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
        console.log("nr of cells to render", length);

        var rows_data_slice_updated = Array.from({ length }, (_, i) => ({
          pos: flat_data_slice_updated_final.pos[i], // position type should be changed from string to float
          base: flat_data_slice_updated_final.base[i],
          accession: flat_data_slice_updated_final.accession[i],
        }));
        // console.log("rows updated !", rows_data_slice_updated);

        vis.svgFocus.selectAll(".x-axis--focus").call(
          vis.xAxisFocus.tickValues(
            vis.xScaleFocus.domain().filter(function(d, i) {
              return !(i % 20); // defines tick interval
            })
          )
        );
        vis.svgFocus.select(".x-axis--focus .domain").remove(); // to disable rendering the axis line

        updateVariantFocusChart(rows_data_slice_updated);

        vis.svgFocus
          .selectAll(".snp-cell")
          .on("mouseover", mouseover)
          .on("mousemove", mousemove)
          .on("mouseleave", mouseleave);
      }

      // !! IMPORTANT Hide y-axis when no VR (this is only when full color is loaded)
      // var newAcc = d3.select("#selectButtonAccessionData").node().value;
      // console.log("selected VR from component sequence: ", newAcc);

      // // if (newAcc === "_full") {
      // if (newAcc === "_ref") {

      //   // console.log("no vr --> axis should NOT be visible");

      //   d3.select(".y-axis--context").style("opacity", "0");
      //   d3.select(".y-axis-title").style("opacity", "0");
      // } else {
      //   // console.log("vr --> axis should be visible");

      //   d3.select(".y-axis--context").style("opacity", "1");
      //   d3.select(".y-axis-title").style("opacity", "1");

      // }

      //Tooltip at the end otherwise <div> not yet loaded
      vis.svgFocus
        .selectAll(".snp-cell")
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave);

      vis.svgContext
        .selectAll(".variants--summary-bar")
        .on("mouseover", mouseoverBar)
        .on("mousemove", mousemoveBar)
        .on("mouseleave", mouseleaveBar);

        vis.phenoChart
        .selectAll(".pheno-cell")
          .on("mouseover", mouseoverPheno)
          .on("mousemove", mousemovePheno)
          .on("mouseleave", mouseleavePheno);

        vis.phenoBars
        .selectAll(".pheno-bar")
        .on("mouseover", mouseoverPhenoBar)
        .on("mousemove", mousemovePhenoBar)
        .on("mouseleave", mouseleavePhenoBar);

        vis.phenoBars2
        .selectAll(".pheno-bar2")
        .on("mouseover", mouseoverPhenoBar2)
        .on("mousemove", mousemovePhenoBar2)
        .on("mouseleave", mouseleavePhenoBar2);


    },
  },
  mounted() {
    // function initVis() {
    let vis = this;

    var globalBrushEnd = 50; 
    vis.globalBrushEnd = globalBrushEnd;
    
    var globalBrushStart = 0;
    vis.globalBrushStart = globalBrushStart;

    // Define dimensions of the visualization
    const containerWidth = d3.select("#chart").node().clientWidth;
    const containerHeight = 500;

    const margin = { top: 5, right: 5, bottom: 0, left: 5 };
    const innerMargin = 12;
    // const tooltipMargin = 10;
    const legendHeight = 40;

    const width = containerWidth - margin.left - margin.right;
    const height = containerHeight - margin.top - margin.left - legendHeight;

    const leftColWidth = (width/10)*2;
    vis.leftColWidth = leftColWidth;
    const variantSumWidth = 50;
    const variantBarcodeChartWidth = leftColWidth - variantSumWidth;
    // const midColWidth = 500;
    const midColWidth = (width/10)*5;
    vis.midColWidth = midColWidth;
    const rightColWidth = width - leftColWidth - midColWidth;
    const phenoBarsWidth = 130;
    const phenoChartWidth = rightColWidth - phenoBarsWidth;
    vis.phenoChartWidth = phenoChartWidth;


    const topRowHeight = 30;
    vis.topRowHeight = topRowHeight;
    const bottomRowHeight = height - topRowHeight - innerMargin * 5;

    // // set dimensions
    // var margin = { top: 30, right: 20, bottom: 40, left: 80 },
    //   width =
    //     (d3.select("#msa_chart").node().clientWidth - margin.left - margin.right)/2,
    //   height = 550 - margin.top - margin.bottom;
    var focusHeight = 30;
    vis.focusHeight = focusHeight;
    vis.margin = margin;
    vis.width = width;
    vis.height = height;

    // set color scale
    var colors = {
      A: "#4daf4a",
      a: "#4daf4a",
      G: "#e41a1c",
      g: "#e41a1c",
      C: "#ff7f00",
      c: "#ff7f00",
      T: "#377eb8",
      t: "#377eb8",
      // "-": "#E6E6E6",
      "-": "rgb(214,218,224)",
      // "*": "rgb(240,240,244)",
      "*": "rgb(39,43,50)",
      // "*": "#686868",
      // "*": "white",
    };
    vis.colors = colors;

    // legendVariants labels
    var dataLabels = [1, 2, 3, 4, 5, 6];
    var cols = [
      "#4daf4a",
      "#ff7f00",
      "#e41a1c",
      "#377eb8",
      // "#E6E6E6",
      "rgb(214,218,224)",
      // "rgb(240,240,244)",
      "rgb(39,43,50)",
      //  "#686868",
      // "white"
    ];
    var bases = ["A", "C", "G", "T", "same", "unknown or gap"];

    // define the accession orders
    var orders = {
      original_map: "original mapping",
      dtf1: "DTF1 descending",
      dtf1_rev: "DTF1 ascending",
      dtf3: "DTF3 descending",
      dtf3_rev: "DTF3 ascending",
      ref_first: "reference accessions first",
      ref_last: "reference accessions last",
      // alpha_asc: "alphabetical",
      // alpha_desc: "alphabetical reversed",
      phylo: "phylogeny",
      phylo_rev: "phylogeny reversed",
      group: "group",
      group_rev: "group reversed",
      origin: "origin",
      origin_rev: "origin reversed",
      
    };
    vis.orders = orders;

    // define the accession orders
    var ordersPheno = {
      original_map: "original mapping",
      dtf1: "DTF1 descending",
      dtf1_rev: "DTF1 ascending",
      dtf3: "DTF3 descending",
      dtf3_rev: "DTF3 ascending",
      ref_first: "reference accessions first",
      ref_last: "reference accessions last",
      // alpha_asc: "alphabetical",
      // alpha_desc: "alphabetical reversed",
      phylo: "phylogeny",
      phylo_rev: "phylogeny reversed",
      group: "group",
      group_rev: "group reversed",
      origin: "origin",
      origin_rev: "origin reversed",
     
    };
    vis.ordersPheno = ordersPheno;

    // define brush sizes
    var brushSizes = {
      1: "100 positions",
      2: "200 positions",
      3: "300 positions",
      4: "400 positions",
    };
    vis.brushSizes = brushSizes;

    
    // initialize scales
    var xScaleContext = d3.scaleLinear().range([0, midColWidth]);
    vis.xScaleContext = xScaleContext;

    var yScaleContext = d3.scaleLinear().range([topRowHeight, 0]);
    vis.yScaleContext = yScaleContext;

    var xScaleFocus = d3
      .scaleBand()
      .range([0, midColWidth])
      .padding(0.14);
    vis.xScaleFocus = xScaleFocus;

    var yScaleFocus = d3
      .scaleBand()
      .range([bottomRowHeight, 0])
      .padding(0.07);
    vis.yScaleFocus = yScaleFocus;

    var xScalePhenos = d3
      .scaleBand()
      .range([0, phenoChartWidth])
      .padding(0.6);
    vis.xScalePhenos = xScalePhenos;

    var yScalePhenos = d3
      .scaleBand()
      .range([bottomRowHeight, 0])
      .padding(0.3)
    vis.yScalePhenos = yScalePhenos;

    var yScalePhenoBars = d3
      .scaleBand()
      .range([bottomRowHeight, 0])
      .padding(0.4)
    vis.yScalePhenoBars = yScalePhenoBars;

    var xScalePhenoBars = d3.scaleLinear()
        .range([0, phenoBarsWidth/2]);
    vis.xScalePhenoBars = xScalePhenoBars;

    var yScalePhenoBars2 = d3
      .scaleBand()
      .range([bottomRowHeight, 0])
      .padding(0.4)
    vis.yScalePhenoBars2 = yScalePhenoBars2;

    var xScalePhenoBars2 = d3.scaleLinear()
        .range([0, phenoBarsWidth/2]);
    vis.xScalePhenoBars2 = xScalePhenoBars2;

    var colorScaleDTF1 = d3.scaleLinear()
      .domain([28, 137])
      .range(['#d3eecd', '#2a8d46'])

    vis.colorScaleDTF1 = colorScaleDTF1;

    var colorScaleDTF3 = d3.scaleLinear()
      .domain([28, 116])
      .range(['#deebf7', '#08519c'])

    vis.colorScaleDTF3 = colorScaleDTF3;

    var colorScaleGroup = d3.scaleOrdinal()
      .domain(['X', 'Y'])
      .range(['black', 'white'])

    vis.colorScaleGroup = colorScaleGroup;

    var colorScaleOrigin = d3.scaleOrdinal()
      .domain(['United States', 'Belgium', 'Portugal', 'Cape Verde', 'Sweden', 'Japan', 'Germany', 'Tajikistan', 'India', 'China'])
      // .range(['#8dd3c7', '#ffffb3', '#bebada', '#fb8072', '#80b1d3', '#fdb462', '#b3de69', '#fccde5', '#d9d9d9', '#bc80bd'])
      //.range(['#e41a1c','#377eb8','#4daf4a','#984ea3','#ff7f00','#ffff33','#a65628','#f781bf','#999999'])
      .range(['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c','#fdbf6f','#ff7f00','#cab2d6','#6a3d9a'])

    vis.colorScaleOrigin = colorScaleOrigin;

    //initialize axes
    var xAxisContext = d3
      .axisBottom()
      .scale(vis.xScaleContext)
      .tickSizeOuter(0);
    vis.xAxisContext = xAxisContext;

    var yAxisContext = d3
      .axisLeft(vis.yScaleContext)
      .ticks(2)
      .tickSizeOuter(0);
    vis.yAxisContext = yAxisContext;

    var xAxisFocus = d3.axisTop(vis.xScaleFocus);
    vis.xAxisFocus = xAxisFocus;

    var yAxisFocus = d3.axisLeft(vis.yScaleFocus);
    vis.yAxisFocus = yAxisFocus;

    var xAxisPhenos = d3.axisTop(vis.xScalePhenos);
    vis.xAxisPhenos = xAxisPhenos;

    var yAxisPhenos = d3.axisRight(vis.yScalePhenos).tickValues([]);
    vis.yAxisPhenos = yAxisPhenos;

    var yAxisPhenoBars = d3.axisRight(vis.yScalePhenoBars).tickValues([]);
    vis.yAxisPhenoBars = yAxisPhenoBars;

    var xAxisPhenoBars = d3
      .axisTop()
      .scale(vis.xScalePhenoBars)
      .ticks(3)
      .tickSizeOuter(0);
    vis.xAxisPhenoBars = xAxisPhenoBars;

    var yAxisPhenoBars2 = d3.axisRight(vis.yScalePhenoBars2).tickValues([]);
    vis.yAxisPhenoBars2 = yAxisPhenoBars2;

    var xAxisPhenoBars2 = d3
      .axisTop()
      .scale(vis.xScalePhenoBars2)
      .ticks(3)
      .tickSizeOuter(0);
    vis.xAxisPhenoBars2 = xAxisPhenoBars2;

    // initalize brush
    var brush = d3.brushX().extent([
      [0, 0],
      [midColWidth, focusHeight],
    ]);
    vis.brush = brush;

    const arrowLeft = d3
    .select("#footer-col-right")
      .attr(
        "transform",
        `translate(${leftColWidth}, ${0})`
      );


    // Prepare the overall layout
    const svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", containerWidth)
      .attr("height", containerHeight)
      .append("g")
      .attr("transform", `translate(${margin.left}, ${margin.top})`);

    vis.svg = svg;

    const svgContext = svg
      .append("g")
      .attr("transform", `translate(${leftColWidth}, 10)`);

    vis.svgContext = svgContext;

    svgContext
      .append("rect")
      .attr("class", "background-gene--context")
      .attr("width", midColWidth)
      .attr("height", topRowHeight);
    // .style("fill", "lightblue");

    // append brush
    svgContext
      .append("g")
      .attr("class", "brush")
      .attr("transform", "translate(0," + 0 + ")");

    const svgContextLabels = svgContext
      .append("g")
      .attr("class", "brushLabels")
      .style("font-size", 11)
      .style("font-family", "sans-serif")
      .style("fill", "cornflowerblue")
      .style("font-weight", 600)
      .attr("transform", `translate(${0}, ${vis.margin.top - 10})`);

    vis.svgContextLabels = svgContextLabels;

    var labelL = vis.svgContextLabels.append("g").attr("class", "brushLabelL");

    vis.labelL = labelL;

    var labelR = vis.svgContextLabels.append("g").attr("class", "brushLabelR");

    vis.labelR = labelR;

    const varBarcodeChart = svg
      .append("g")
      .attr("transform", `translate(0, ${topRowHeight + innerMargin * 6})`);

    vis.varBarcodeChart = varBarcodeChart;

    vis.varBarcodeChart
      .append("rect")
      .attr("class", "background-summary")
      .attr("width", variantBarcodeChartWidth)
      .attr("height", bottomRowHeight)
      .style("fill", "lightgrey")
      .style("opacity", "0.2");

    const phenoChart = svg
      .append("g")
      .attr(
        "transform",
        `translate(${leftColWidth + midColWidth}, ${topRowHeight +
          innerMargin * 6})`
      );

    vis.phenoChart = phenoChart;

    vis.phenoChart
      .append("rect")
      .attr("class", "background-summary")
      .attr("width", phenoChartWidth/2)
      .attr("height", bottomRowHeight)
      .style("fill", "white")
      .style("opacity", "1");

      const phenoBars = svg
      .append("g")
      .attr(
        "transform",
        `translate(${leftColWidth + midColWidth + phenoChartWidth + innerMargin/2 + phenoBarsWidth/2}, ${topRowHeight +
          innerMargin * 6})`
      );

    vis.phenoBars = phenoBars;

    vis.phenoBars
      .append("rect")
      .attr("class", "background-summary-phenobar")
      .attr("width", phenoBarsWidth/2)
      .attr("height", bottomRowHeight)
      .style("fill", "steelblue")
      .style("opacity", "0.1");


      const phenoBars2 = svg
      .append("g")
      .attr(
        "transform",
        `translate(${leftColWidth + midColWidth + phenoChartWidth}, ${topRowHeight +
          innerMargin * 6})`
      );

    vis.phenoBars2 = phenoBars2;

    vis.phenoBars2
      .append("rect")
      .attr("class", "background-summary-phenobar")
      .attr("width", phenoBarsWidth/2)
      .attr("height", bottomRowHeight)
      .style("fill", "#2a8d46")
      .style("opacity", "0.1");



    const svgFocus = svg
      .append("g")
      .attr(
        "transform",
        `translate(${leftColWidth}, ${topRowHeight + innerMargin * 6})`
      );

    vis.svgFocus = svgFocus;

    // append empty axis groups
    var xAxisContextG = vis.svgContext
      .append("g")
      .attr("class", "x-axis--context")
      .style("font-size", 10)
      .attr("transform", "translate(0," + focusHeight + ")");
    vis.xAxisContextG = xAxisContextG;

    // append empty axis groups
    var yAxisContextG = vis.svgContext
      .append("g")
      .attr("class", "y-axis--context")
      .style("font-size", 10)
      .attr("transform", "translate(0," + 0 + ")");
    vis.yAxisContextG = yAxisContextG;

    // Append both axis titles
    vis.svgContext
      .append("text")
      .attr("class", "x-axis-title")
      .attr("y", vis.margin.top + vis.focusHeight + innerMargin * 2)
      .attr("x", midColWidth / 2)
      .attr("dy", "0.5em")
      .style("text-anchor", "middle")
      .style("font-size", 10)
      .style("font-family", "sans-serif")
      // .style("font-weight", 500)
      .text("Length Gene");

    vis.svgContext
      .append("text")
      .attr(
        "transform",
        `translate(${-80}, ${vis.focusHeight / 2 - 5}) rotate(-90)`
      )
      .attr("class", "y-axis-title")
      .attr("y", vis.focusHeight + innerMargin)
      .attr("x", -10)
      .attr("dy", "0.5em")
      // .attr("transform", "rotate(270)")
      .style("text-anchor", "middle")
      .style("font-size", 10)
      .style("font-family", "sans-serif")
      .text("# SNPs");

    // Append both axis titles
    vis.phenoBars
      .append("text")
      .attr("class", "x-axis-title")
      .attr("y", -30)
      .attr("x", phenoBarsWidth / 4)
      .attr("dy", "0.5em")
      .style("text-anchor", "middle")
      .style("font-size", 10)
      .style("font-family", "sans-serif")
      // .style("font-weight", 500)
      .text("DTF3");

    // Append both axis titles
    vis.phenoBars2
      .append("text")
      .attr("class", "x-axis-title")
      .attr("y", -30)
      .attr("x", phenoBarsWidth / 4)
      .attr("dy", "0.5em")
      .style("text-anchor", "middle")
      .style("font-size", 10)
      .style("font-family", "sans-serif")
      // .style("font-weight", 500)
      .text("DTF1");

    var xAxisFocusG = vis.svgFocus
      .append("g")
      .attr("class", "x-axis--focus")
      .style("font-size", 10);
    // .attr("transform", "translate(0," + this.height + ")")
    vis.xAxisFocusG = xAxisFocusG;

    var yAxisFocusG = vis.svgFocus
      .append("g")
      .attr("class", "y-axis--focus")
      .style("font-size", 10);
    vis.yAxisFocusG = yAxisFocusG;

    var xAxisPhenosG = vis.phenoChart
      .append("g")
      .attr("class", "x-axis--phenos")
      .style("font-size", 10);
    // .attr("transform", "translate(0," + this.height + ")")
    vis.xAxisPhenosG = xAxisPhenosG;

    var yAxisPhenosG = vis.phenoChart
      .append("g")
      .attr("class", "y-axis--phenos")
      .style("font-size", 10);
    vis.yAxisPhenosG = yAxisPhenosG;

    var xAxisPhenoBarsG = vis.phenoBars
      .append("g")
      .attr("class", "x-axis--phenobars")
      .style("font-size", 10);
    // .attr("transform", "translate(0," + this.height + ")")
    vis.xAxisPhenoBarsG = xAxisPhenoBarsG;

    var yAxisPhenoBarsG = vis.phenoBars
      .append("g")
      .attr("class", "y-axis--phenobars")
      .style("font-size", 10);
    vis.yAxisPhenoBarsG = yAxisPhenoBarsG;

    var xAxisPhenoBarsG2 = vis.phenoBars2
      .append("g")
      .attr("class", "x-axis--phenobars2")
      .style("font-size", 10);
    // .attr("transform", "translate(0," + this.height + ")")
    vis.xAxisPhenoBarsG2 = xAxisPhenoBarsG2;

    var yAxisPhenoBarsG2 = vis.phenoBars2
      .append("g")
      .attr("class", "y-axis--phenobars2")
      .style("font-size", 10);
    vis.yAxisPhenoBarsG2 = yAxisPhenoBarsG2;

    // append legend
    var legendVariants = svgFocus
      .selectAll(".legendVariants")
      .data(dataLabels)
      .enter()
      .append("g")
      .attr("class", "legendVariants")
      .attr("transform", function(d, i) {
        return (
          "translate(" +
          66 * i +
          "," +
          (topRowHeight + bottomRowHeight - 20) +
          ")"
        );
      });

    legendVariants
      .append("rect")
      .attr("width", 15)
      .attr("height", 15)
      .attr("rx", 1.5)
      .attr("ry", 1.5)
      .style("fill", function(d, i) {
        return cols[i];
      })
      .style("opacity", 0.8);

    legendVariants
      .append("text")
      .attr("x", 18)
      .attr("y", 8)
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

      // add the options to the button
    d3.select("#selectButtonSortPhenos")
      .selectAll("myOptions")
      .data(Object.keys(ordersPheno))
      .enter()
      .append("option")
      .text(function(d) {
        return ordersPheno[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value returned by the button
    // console.log("orders", Object.keys(orders));

    // // set default option
    // d3.select("#selectButtonBrush").property("value", 1);

    // create a tooltip
    // var tooltip = d3
    //   .select("#msa_chart")
    //   .append("div")
    //   .attr("class", "tooltip")
    //   .style("background-color", "black")
    //   .style("opacity", 0)
    //   .style("border", "solid")
    //   .style("border-width", "0px")
    //   .style("border-radius", "3px")
    //   .style("padding", "5px");
    // vis.tooltip = tooltip;

    // }
    // initVis();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
/* #tooltip {
  position: absolute;
  display: none;
  background-color: black;
  border: solid;
  border-width: 0px;
  border-radius: 3px;
  padding: 5px;
  text-align: left;
} */

#tooltip {
  position: absolute;
  opacity: 0;
  background: #fff;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid #ddd;
  font-size: 0.8rem;
  font-weight: 600;
  padding: 2px 8px;
  text-align: left;
}

#tooltip-bars {
  position: absolute;
  opacity: 0;
  background: #fff;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid #ddd;
  font-size: 0.8rem;
  font-weight: 600;
  padding: 2px 8px;
  text-align: left;
}

#tooltip-pheno {
  position: absolute;
  opacity: 0;
  /* background: #fff; */
  background: black;
  color: white;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  /* border: 1px solid #ddd; */
  border: 1px solid black;
  font-size: 0.8rem;
  font-weight: 600;
  padding: 2px 8px;
  text-align: left;
}

#tooltip-bars--pheno {
  position: absolute;
  opacity: 0;
  background: black;
  color: white;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid black;
  font-size: 0.8rem;
  font-weight: 600;
  padding: 2px 8px;
  text-align: left;
}

#tooltip-bars--pheno2 {
  position: absolute;
  opacity: 0;
  background: black;
  color: white;
  box-shadow: 2px 2px 3px 0px rgb(92 92 92 / 0.5);
  border: 1px solid black;
  font-size: 0.8rem;
  font-weight: 600;
  padding: 2px 8px;
  text-align: left;
}


.selectButtonVariants {
  margin-left: 5px;
  margin-right: 20px;
}

.selectLabelVariants {
  display: inline;
  float: left;
  margin-left: 240px;
  margin-top: 0px;
  font-weight: 700;
}

.background-gene--context {
  /* fill: slategray; */
  /* fill: #8da0cb; */
  fill: #e7298a;
  fill-opacity: 0.06;
}

/* .variants--summary {
  stroke: black;
  stroke-width: 1.5;
  opacity: 0.2;
} */

.variants--summary-bar {
  /* fill: rgb(190, 99, 150); */
  fill: #e7298a;
  /* fill: #8da0cb; */
  opacity: 0.8;
}

.selection {
  stroke: cornflowerblue;
  fill: cornflowerblue;
  fill-opacity: 0.3;
}

#btn-arrow-left {
  margin-left: 120px;
} 

#btn-arrow-right {
  margin-right: 240px;
} 

#header-arrows {
  padding-bottom: 5px;
}

#sort_group {
  margin-left: 110px;
}



</style>
