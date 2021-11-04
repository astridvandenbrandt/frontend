<template>
  <div id="app">
    <div class="container-fluid">
      <div class="min-vh-100">
        <!-- <div class="row">
          <div class="col-2">
            <a href="https://docs.google.com/forms/d/e/1FAIpQLSdE9VeqzUMYN5ZnWicPdOLSfYOdD8OhFHiTxBt-lAMWdEEX4w/viewform?usp=sf_link" class="btn btn-warning btn-sm" role="button" aria-pressed="true"> 
              Feedback Survey</a>
          </div>
          </div> -->
        <div class="row gx-2">
          <div class="col-md-2">
            <div class="sidebar-section">
              <div class="sidebar-item" id="sidebar-menu-header">
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
              <!-- <div class="sidebar-item">
                <p>Select Phenotypes</p>
              </div> -->
            </div>
            <div class="content-section" id="tree-comp">
              <div class="content-title">
                Species Tree
              </div>
              <TreeVis ref="tree_data" />
            </div>
          </div>

          <!-- <div class="col-md-2">
            <div class="content-section"> -->
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <!-- <div class="content-title">
                <p>Matrix dragging</p>
              </div> -->
              <!-- <matrixDrag/> -->
            <!-- </div>
          </div> -->
          
          <div class="col-md-10">
            <!-- <div class="content-section"> -->
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <!-- <div class="content-title">
                <p>Variant Summary</p>
              </div>
            </div> -->
            <div class="content-section">
              <!-- <VariantOverview ref="variant_data_overview" /> -->
              <div class="content-title">
                Gene Sequences and Phenotypes
              </div>
              <GeneVariantVis ref="variant_data" />
            </div>
          </div>
          <!-- <div class="col-md-2"> -->
            <!-- <div class="content-section">
              <div class="content-title">
                <p>Sequence Similarity</p>
              </div>
            </div> -->
           
            <!-- <div class="content-section">
              <div class="content-title">
                <p>Matrix rows example</p>
              </div>
              <matrixDrag/>
            </div> -->
          <!-- </div> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
import TreeVis from "./components/TreeVis.vue";
import GeneVariantVis from "./components/GeneVariantVis.vue";
// import matrixDrag from "./components/matrixDrag.vue";

export default {
  name: "App",
  components: {
    TreeVis,
    GeneVariantVis,
    // matrixDrag
  },
  data() {
    return {
      loadDataMSA: {},
      loadDataMut: {},
      loadDataTree: {},
      loadDataPheno: {}
    };
  },
  mounted() {
    console.log("App loaded");

    // Gene Ids
    const geneIDs = {
      AT1G01010_1: "AT1G01010",
      AT1G01020_1: "AT1G01020",
      AT1G01030_1: "AT1G01030",
      AT1G01040_1: "AT1G01040",
      AT1G01050_1: "AT1G01050",
      AT1G01060_1: "AT1G01060",
      AT1G11460_1: "AT1G11460",
      AT1G02820_1: "AT1G02820",
      AT5G47700_1: "AT5G47700",

    };

    const accessionNames = {
      AT1G01010_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G01020_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G01030_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G01040_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G01050_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G01060_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT1G11460_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_7_Ler", name: "7_Ler"},
      ],
      AT1G02820_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi", name: "4_Cvi"},
        {ref:"_5_Eri", name: "5_Eri"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha", name: "8_Sha"},
      ],
      AT5G47700_1: [
        {ref : "_ref", name: "none"},
        {ref: "_1_Col-0", name:"1_Col-0"},
        {ref:"_2_An-1", name: "2_An-1"},
        {ref:"_3_C24", name:  "3_C24"},
        {ref:"_4_Cvi(1)", name: "4_Cvi(1)"},
        {ref:"_4_Cvi(2)", name: "4_Cvi(2)"},
        {ref:"_5_Eri(1)", name: "5_Eri(1)"},
        {ref:"_5_Eri(2)", name: "5_Eri(2)"},
        {ref:"_6_Kyo", name: "6_Kyo"},
        {ref:"_7_Ler", name: "7_Ler"},
        {ref:"_8_Sha(1)", name: "8_Sha(1)"},
        {ref:"_8_Sha(2)", name: "8_Sha(2)"},
      ],
    };
    console.log('accession list per gene', accessionNames)
    this.accessionNames = accessionNames;

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

     
     const accessionIDs = {
      // _full_closest: "none",
      _ref: "none",
      "_1_Col-0": "1_Col-0",
      "_2_An-1": "2_An-1",
      "_3_C24":  "3_C24",
      "_4_Cvi": "4_Cvi",
      "_5_Eri": "5_Eri",
      "_6_Kyo": "6_Kyo",
      "_7_Ler": "7_Ler",
      "_8_Sha": "8_Sha",
      // "_Altai-5": "Altai-5",
      // "_Gro-3": "Gro-3",
      // "_Kas-1": "Kas-1",
      // "_Ler-0": "Ler-0",
      // "_Sku-30": "Sku-30",
      // "_Tsu-0": "Tsu-0"

    };
    // console.log('accession ids original', accessionIDs)

  

    this.selectedGeneId = d3.select("#selectButtonData").node().value;
    console.log('selected gene ID', this.selectedGeneId)

    var accessionsGene = accessionNames[this.selectedGeneId];
    console.log('selected gene accessions', accessionsGene)
    this.accessionsGene = accessionsGene;

    let accessionButton = d3.select("#selectButtonAccessionData");
    this.accessionButton = accessionButton;

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

    // var accessionTest = this.loadDataAccession;
    // console.log('accession test', accessionTest)
  },
  methods: {
    updateData() {

      let vis = this;

      //add the options to the button
      // var accessionButton = d3.select("#selectButtonAccessionData")
      vis.accessionButton
        .selectAll("myOptionsAccessionData")
        .data(vis.accessionsGene) //Object.keys(accessionIDs)
        .enter()
        .append("option")
        .attr('class', 'myOptionsAccessionData')
        .text(d => d.name)
        .attr("value", d => d.ref);
        // .text(function(d) {
        //   // return accessionIDs[d];
        //   console.log('d in list name', d.name) //accessionsGene[d]
        //   return d.name; //accessionsGene[d]
        // }) // text showed in the menu
        // .attr("value", function(d) {
        //   console.log('d in list ref', d.ref) //accessionsGene[d]
        //   return d.ref;
        // }); // corresponding value returned by the button

      // vis.accessionButton = accessionButton;
      // console.log("vis accession button inital", this.accessionButton)
    
      // Change data based on select
      d3.select("#selectButtonData").on("change", () => {
        var selectedGene = d3.select("#selectButtonData").node().value;

        console.log("TEST!!");

        var selectedAcc = vis.accessionNames[selectedGene];
        console.log("selected gene ID accessions in update: ", selectedAcc);

        // check new data
        let accessionButtonValueUpdate = vis.accessionButton
            // .select("#selectButtonAccessionData")
            .selectAll(".myOptionsAccessionData")
            // .selectAll(".accessionOption")
            .data(selectedAcc); //key function?

          console.log("vis accession button update", accessionButtonValueUpdate)

        
          // make new cells
          let accessionButtonValueEnter = accessionButtonValueUpdate
            .enter()
            .append("option")
            .attr('class', 'myOptionsAccessionData')

          // remove old cells
          accessionButtonValueUpdate.exit().remove();

          console.log("vis accession button enter", accessionButtonValueEnter)

          // merge cells with existing
          accessionButtonValueEnter
            .merge(accessionButtonValueUpdate)
            .text(d => d.name)
            .attr("value", d => d.ref); // corresponding value returned by the button
          console.log("vis accession button enter final", accessionButtonValueEnter)


        // // check new data
        // let accessionButtonTextUpdate = vis.accessionButton
        //     // .select("#selectButtonAccessionData")
        //     .selectAll(".myOptionsAccessionData")
        //     // .selectAll(".accessionOption")
        //     .data(selectedAcc); //key function?

        //   console.log("vis accession button update", accessionButtonTextUpdate)

        
        //   // make new cells
        //   let accessionButtonTextEnter = accessionButtonValueUpdate
        //     .enter()
        //     .append("option")
        //     .attr('class', 'myOptionsAccessionData')

        //   // remove old cells
        //   accessionButtonValueUpdate.exit().remove();

        //   console.log("vis accession button enter", accessionButtonValueEnter)

        //   // merge cells with existing
        //   accessionButtonValueEnter
        //     .merge(accessionButtonValueUpdate)
        //     // .text(d => d.name)
        //     .attr("value", d => d.ref); // corresponding value returned by the button
        //   console.log("vis accession button enter final", accessionButtonValueEnter)
        




        var selectedAccession = d3.select("#selectButtonAccessionData").node().value;
        // console.log("selected Accession: ", selectedAccession);

        this.fetchData(selectedGene, selectedAccession);
      });

      // Change data based on select
      d3.select("#selectButtonTreeData").on("change", () => {
        var selectedTree = d3.select("#selectButtonTreeData").node().value;
        // console.log("selected Tree Type: ", selectedTree);

        this.fetchDataTree(selectedTree);
      });
      // Change data based on select
      d3.select("#selectButtonAccessionData").on("change", () => {
        var selectedGene = d3.select("#selectButtonData").node().value;
        // console.log("selected gene ID: ", selectedGene);

        var selectedAccession = d3.select("#selectButtonAccessionData").node().value;
        // console.log("selected Accession: ", selectedAccession);

        this.fetchData(selectedGene, selectedAccession);
      });
    },

    async fetchDataTree(treeType) {
      //change later to type of tree
      // console.log("initial Tree =", treeType);

      const response = await fetch("./trees/tree_" + treeType + ".txt");
      const data_tree = await response.text();
      // console.log(data_tree);

      this.loadDataTree = data_tree;
      let componentTree = this.$refs["tree_data"];

      componentTree.updateVis(data_tree);
    },

    // async fetchDataAccession(geneID) {
    //   // console.log("this.geneID =", geneID);
    //   // console.log("initial Accession =", accession);    

    //   let data_accession = await d3.csv("./gene_variants/accessions_" + geneID + ".csv");
    //   this.loadDataAccession = data_accession;

    //   // console.log('data accessions in App', data_accession)

    // },

    async fetchData(geneID, accession) {
      console.log("initial Gene =", geneID);
      console.log("initial Accession =", accession);    

      let data_msa = await d3.csv("./gene_variants_newest/matrix_" + geneID + accession + ".csv");
      this.loadDataMSA = data_msa;

      let data_msa_mutations = await d3.csv(
        "./gene_variants_newest/var_count_matrix_" + geneID + accession +".csv"
      ); //  msa_AT1G01060_mutations.csv
      this.loadDataMut = data_msa_mutations;

      // let data_barcode = await d3.csv(
      //   "./gene_variants_newest/barcode_" + geneID + accession + ".csv"
      // ); //  msa_AT1G01060_mutations.csv
      // this.loadDataBarcode = data_barcode;

      let data_phenos = await d3.csv(
        "./gene_variants_newest/pheno_" + geneID +  ".csv"
      ); 
      this.loadDataPheno = data_phenos;

      let componentVariantDetails = this.$refs["variant_data"];

      componentVariantDetails.updateVis(data_msa, data_msa_mutations, data_phenos);
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
  text-transform: uppercase;
  font-size: 13px;
  text-align: left;
  padding-left: 1%;
  font-family: sans-serif;
  /* font-variant: all-small-caps; */
  font-weight: 550;
  letter-spacing: 0.08em;
}

/* .content-title p {
  text-transform: uppercase;
  font-weight: 600;
  text-align: left;
  margin-left: 1%;
} */

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
  border-top: solid 1px rgba(39, 39, 39, 0.178);
}

#sidebar-menu-header {
  border-top: 0;

}

#tree-comp{
  margin-top: 30px;
}

p {
   -webkit-appearance: none;
   -moz-appearance: none;
   appearance: none;

 }

</style>
