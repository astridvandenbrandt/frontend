<template>
  <div id="app">
    <div class="container-fluid">
      <div class="row">
        <div class="col-md-5">
          <TreeVis />
        </div>
        <div class="col-md-7">
            
          <VariantOverview ref="variant_data_overview" />
          <VariantDetail ref="variant_data" />
        </div>
          

        </div>
      </div>
      <div class="row">
        <div class="col-md-5">
          <!-- <BrushExample ref="brush_data" />    -->
          <EnterUpdateExample ref="alpha_data"/>   
          <!-- <EnterUpdateExample/>   -->
           <!-- <BrushZoomExample ref="brush_data" />   -->
             
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
import TreeVis from "./components/TreeVis.vue";
import VariantDetail from "./components/VariantDetail.vue";
import VariantOverview from "./components/VariantOverview.vue";
// import BrushExample from "./components/BrushExample.vue";
import EnterUpdateExample from "./components/EnterUpdateExample.vue";
// import BrushZoomExample from "./components/BrushZoomExample.vue";

export default {
  name: "App",
  components: {
    TreeVis,
    VariantDetail,
    VariantOverview,
    // BrushExample,
    EnterUpdateExample
    // BrushZoomExample
  },
  data() {
    return {
      loadData: {},
    };
  },
  mounted() {
    console.log("App loaded");
    this.fetchData();
  },
  methods: {
    async fetchData() {
      // let data_heatmap_sort = await d3.tsv("./data2.tsv", function(d) {
      //   return {
      //     row: +d.row_idx,
      //     col: +d.col_idx,
      //     value: +d.log2ratio,
      //   };
      // });
      // this.loadData = data_heatmap_sort;
      // let componentVariantDetailsSort = this.$refs["variant_data_sort"];
      // componentVariantDetailsSort.updateVis(data_heatmap_sort);

      let data_heatmap = await d3.csv("./slice_gene_2.csv");
      this.loadData = data_heatmap;
      let data_heatmap_copy = await d3.csv("./heatmap_data_copy.csv");
      this.loadData = data_heatmap_copy;
      // let data_brush = await d3.csv("./aapl.csv")
      // this.loadData = data_brush;
      let alphabet = await d3.csv("./alphabet.csv");
      this.loadData = alphabet;
    
      let componentVariantDetails = this.$refs["variant_data"];
      let componentVariantOverview = this.$refs["variant_data_overview"];
      // let componentBrushExample = this.$refs["brush_data"];
      let componentEnterUpdate = this.$refs["alpha_data"];
      componentVariantDetails.updateVis(data_heatmap);
      componentVariantOverview.updateVis(data_heatmap_copy);
      // componentBrushExample.updateVis(data_heatmap_copy);
      componentEnterUpdate.updateVis(alphabet);
      // componentBrushExample.updateVis(data_brush);
      

    },
  },
  // async mounted() {
  //   const data_names = await d3.csv("/test.csv");
  //   for (const { name, age } of data_names) {
  //     console.log('TEST CSV', name, age);
  //   }
  //   const data_tsv = await d3.tsv("/data.tsv");
  //   for (const { day, hour} of data_tsv) {
  //     console.log('TEST TSV', day, hour);
  //   }
  // }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
