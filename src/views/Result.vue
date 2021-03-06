<template>
<div class="container">
  <v-row>
    <v-col>
      <h2 style="white-space: nowrap;"> Alignment Result </h2>
    </v-col>
    <div class="float-md-right py-2">
      <v-btn class="mx-2 mr-0" color="orange" elevation="2" @click="download" v-if="result.length > 0">    
        <v-icon dark> mdi-download </v-icon>
        Download alignment as .json ({{fileSize}} kb)
      </v-btn>
    
      <v-dialog v-model="dialog" persistent max-width="290">
        <template v-slot:activator="{ on, attrs }">
          <v-btn class="ml-2" color="orange" v-bind="attrs" v-on="on">
            <v-icon dark> mdi-arrow-left </v-icon>
            Compute new alignment
          </v-btn>
        </template>
        <v-card>
          <v-card-title class="headline">
            Go back?
          </v-card-title>
          <v-card-text> When going back to make a new alignment the results will be deleted.</v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="orange" @click="reset" >
              <v-icon dark> mdi-arrow-left </v-icon>
              Ok
            </v-btn>
            <v-btn color="orange" @click="dialog = false" >
              <v-icon dark> mdi-cancel </v-icon>
              Cancel
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
  </v-row>
  <v-row justify="center">
    <v-col>
      <p> The results are presented in the table. You can download the alignment results as a <code>.json</code>-file. </p>
    </v-col>
  </v-row>
  <v-row>
    <v-col>
      <v-data-table
        :headers="tableHeaders"
        :items="result"
        :items-per-page="10"
        class="elevation-4"
        :footer-props="{'items-per-page-options': [10, -1]}"
      >
        <template v-slot:[`header.relation`]="{ header }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <span
                v-bind="attrs"
                v-on="on"
                style="border-bottom:1px dotted"
              >
                {{header.text}}</span>
            </template>
            <div style="max-width: 300px; text-align: justify;">
              <kbd>=</kbd> indicates that the source element and the target element are semantically equivalent, i.e. they are synonymous. <br>
              <kbd>&lt;</kbd> indicates that the source element is less general than the target element. <br>
              <kbd>&gt;</kbd> indicates that the source element is more general than the target element.
            </div>
          </v-tooltip>
        </template>
         <template v-slot:[`header.confidence`]="{ header }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <span
                v-bind="attrs"
                v-on="on"
                style="border-bottom:1px dotted"
              >
                {{header.text}}</span>
            </template>
            <div style="max-width: 300px; text-align: justify;">
              The confidence value ranges from 0.0 (lowest) to 1.0 (highest) and is a measure stating how confident the system is on the computed relation being correct.
            </div>
          </v-tooltip>
        </template>
        <template v-slot:no-data>
          <v-row>
            <v-col>
              <img 
                src="@/assets/undraw_no_data.svg" 
                alt="Graphic depicting 'no data'"
                style="max-height: 250px;"
                class="mb-4 mt-8"
              >
              <h1>No semantic relations found!</h1>
              <p>We found no semantic relations between the source- and target files. Are you sure the files are related?</p>

            </v-col>
          </v-row>
        </template>
      </v-data-table>
    </v-col>
  </v-row>
</div>
</template>

<script>
export default {
  props: {
    result: {
      type: Array,
      required: true,
    },
    sourceFilename: {
      type: String,
      default: '',
    },
    targetFilename: {
      type: String,
      default: '',
    },
  },
  data: () => ({
    tableHeaders: [
      { text: 'Source', value: 'source' },
      { text: 'Target', value: 'target' },
      { text: 'Relation', value: 'relation' },
      { text: 'Confidence', value: 'confidence' },
    ],
    dialog: false,
    justify: [ 'space-between' ],
  }),
  computed: {
    dataString() {
      return "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.result, undefined, 2))
    },
    fileSize() {
      // ref: https://stackoverflow.com/a/52254083/11192976
      return ((new Blob([this.dataString]).size)/1000).toFixed(1)
    }
  },
  methods: {
    reset() {
      this.$emit('reset')
    },
    download() {
      // ref: https://stackoverflow.com/a/30800715/11192976
      const downloadAnchorNode = document.createElement('a');
      downloadAnchorNode.setAttribute("href", this.dataString);
      downloadAnchorNode.setAttribute("download", `${this.sourceFilename}_${this.targetFilename}_alignment.json`);
      document.body.appendChild(downloadAnchorNode); // required for firefox
      downloadAnchorNode.click();
      downloadAnchorNode.remove();
    }
  }

}
</script>

<style>
</style>
