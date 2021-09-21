<template>
  <div id="app">
    <div class="container-fluid">
      <div class="min-vh-100">
        <div class="row gx-2">
          <div class="col-md-2">
            <div class="sidebar-section">
              <div class="sidebar-item">
                <h4>Menu</h4>
              </div>
              <div class="sidebar-item">
                <label for="selectButtonData"> Gene: </label>
                <select class="selectButtonVariants" id="selectButtonData">
                </select>
              </div>
              <div class="sidebar-item">
                <label for="selectButtonData"> Visual Reference: </label>
                <select class="selectButtonVariants" id="selectButtonAccessionData">
                </select>
              </div>
              <div class="sidebar-item">
                <label for="selectButtonData"> Tree Type: </label>
                <select class="selectButtonVariants" id="selectButtonTreeData">
                </select>
              </div>
              <div class="sidebar-item">
                <p>Select Phenotypes</p>
              </div>
            </div>
          </div>

          <div class="col-md-2">
            <div class="content-section">
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <div class="content-title">
                <p>Variant Summary</p>
              </div>
            </div>
          </div>
          
          <div class="col-md-5">
            <!-- <div class="content-section"> -->
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <!-- <div class="content-title">
                <p>Variant Summary</p>
              </div>
            </div> -->
            <div class="content-section">
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <div class="content-title">
                <p>Gene Sequences</p>
              </div>
              <GeneVariantVis ref="variant_data" />
            </div>
          </div>
          <div class="col-md-3">
            <!-- <div class="content-section">
              <div class="content-title">
                <p>Sequence Similarity</p>
              </div>
            </div> -->
            <div class="content-section">
              <div class="content-title">
                <p>Phylogenetic Tree</p>
              </div>
              <TreeVis ref="tree_data" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
import TreeVis from "./components/TreeVis.vue";
import GeneVariantVis from "./components/GeneVariantVis.vue";

export default {
  name: "App",
  components: {
    TreeVis,
    GeneVariantVis,
    // VariantOverview,
  },
  data() {
    return {
      loadDataMSA: {},
      loadDataMut: {},
      loadDataTree: {},
    };
  },
  mounted() {
    console.log("App loaded");

    // Gene Ids
    var geneIDs = {
      AT1G01060: "AT1G01060",
      AT1G02820: "AT1G02820",
    };

    // add the options to the button
    d3.select("#selectButtonData")
      .selectAll("myOptionsData")
      .data(Object.keys(geneIDs))
      .enter()
      .append("option")
      .text(function(d) {
        return geneIDs[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value returned by the button

     // Gene Ids
     var accessionIDs = {
      _full: "None",
      "_1_Col-0": "1_Col-0",
      "_2_An-1": "2_An-1",
      "_3_C24":  "3_C24",
      "_4_Cvi": "4_Cvi",
      "_5_Eri": "5_Eri",
      "_6_Kyo": "6_Kyo",
      "_7_Ler": "7_Ler",
      "_8_Sha": "8_Sha",
      "_Altai-5": "Altai-5",
      "_Gro-3": "Gro-3",
      "_Kas-1": "Kas-1",
      "_Ler-0": "Ler-0",
      "_Sku-30": "Sku-30",
      "_Tsu-0": "Tsu-0"

    };

    // add the options to the button
    d3.select("#selectButtonAccessionData")
      .selectAll("myOptionsAccessionData")
      .data(Object.keys(accessionIDs))
      .enter()
      .append("option")
      .text(function(d) {
        return accessionIDs[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value returned by the button

    // Trees
    var treeTypes = {
      kmer_distance: "k-mer distance",
      gene_distance: "gene distance",
      ani: "ANI",
      // coresnp: "core SNP tree",
     
    };

    // add the options to the button
    d3.select("#selectButtonTreeData")
      .selectAll("myOptionsTreeData")
      .data(Object.keys(treeTypes))
      .enter()
      .append("option")
      .text(function(d) {
        return treeTypes[d];
      }) // text showed in the menu
      .attr("value", function(d) {
        return d;
      }); // corresponding value

    this.fetchData(Object.keys(geneIDs)[0], Object.keys(accessionIDs)[0]); // inital data display
    this.fetchDataTree(Object.keys(treeTypes)[0]);
    this.updateData();
  },
  methods: {
    updateData() {

      // Change data based on select
      d3.select("#selectButtonData").on("change", () => {
        var selectedGene = d3.select("#selectButtonData").node().value;
        console.log("selected gene ID: ", selectedGene);

        var selectedAccession = d3.select("#selectButtonAccessionData").node().value;
        console.log("selected Accession: ", selectedAccession);

        this.fetchData(selectedGene, selectedAccession);
      });

      // Change data based on select
      d3.select("#selectButtonTreeData").on("change", () => {
        var selectedTree = d3.select("#selectButtonTreeData").node().value;
        console.log("selected Tree Type: ", selectedTree);

        this.fetchDataTree(selectedTree);
      });
      // Change data based on select
      d3.select("#selectButtonAccessionData").on("change", () => {
        var selectedGene = d3.select("#selectButtonData").node().value;
        console.log("selected gene ID: ", selectedGene);

        var selectedAccession = d3.select("#selectButtonAccessionData").node().value;
        console.log("selected Accession: ", selectedAccession);

        this.fetchData(selectedGene, selectedAccession);
      });
    },
    async fetchDataTree(treeType) {
      //change later to type of tree
      console.log("initial Tree =", treeType);

      const response = await fetch("./trees/tree_" + treeType + ".txt");
      const data_tree = await response.text();
      // console.log(data_tree);

      this.loadDataTree = data_tree;
      let componentTree = this.$refs["tree_data"];

      componentTree.updateVis(data_tree);
    },
    async fetchData(geneID, accession) {
      // console.log("initial Gene =", geneID);
      // console.log("initial Accession =", accession);
    

      let data_msa = await d3.csv("./gene_variants/matrix_" + geneID + accession + ".csv");
      this.loadDataMSA = data_msa;

      let data_msa_mutations = await d3.csv(
        "./gene_variants/var_count_" + geneID + accession +".csv"
      ); //  msa_AT1G01060_mutations.csv
      this.loadDataMut = data_msa_mutations;

      let componentVariantDetails = this.$refs["variant_data"];

      componentVariantDetails.updateVis(data_msa, data_msa_mutations);
    },
  },
};
</script>

<style>
#app {
  font-family: Karla, Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 5px;

  /* background: rgba(246, 246, 246, 1); */
  /*  border-radius: 12px; */
  padding: 5px 0 0px;
}

.sidebar-item {
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.content-title {
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgb(246, 247, 249);
}

.content-title p {
  text-transform: uppercase;
  font-weight: 600;
  text-align: left;
  margin-left: 1%;
}

.content-section {
  background: rgba(255, 255, 255, 1);
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1), 0 3px 10px 0 rgba(0, 0, 0, 0.1);
  /* border-radius: 2px; */
}

.sidebar-item {
  text-align: left;

  max-width: 100%;
  margin: 10 auto;
  padding: 10px 0 30px;
  border-bottom: solid 1px rgba(39, 39, 39, 0.178);
}
</style>
