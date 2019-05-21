<template>
  <b-container class="home" fluid>
    <div>
      <b-card no-body>
        <b-tabs pills card vertical>
          <!-- BYTES32 TO STRING -->
          <b-tab :title="options.stringToBytes32" active>
            <b-card-text>
              <b-row align-v="center">
                <b-col md="5">
                  <label class="float-left text-uppercase font-weight-bold">String</label>
                  <b-form-textarea
                    id="textarea"
                    v-model="string"
                    placeholder="String..."
                    rows="6"
                    max-rows="6"
                    @input="setBytes32"
                  ></b-form-textarea>
                </b-col>

                <b-col md="2">
                  <b-button
                    class="m-3"
                    variant="success"
                    @click="saveRecord('bytes32', bytes32, 'string', string)"
                  >Save</b-button>
                </b-col>

                <b-col md="5">
                  <label class="float-left text-uppercase font-weight-bold">Bytes32</label>
                  <b-form-textarea
                    id="textarea"
                    v-model="bytes32"
                    placeholder="bytes32..."
                    rows="6"
                    max-rows="6"
                    @input="setString"
                  ></b-form-textarea>
                </b-col>
              </b-row>
            </b-card-text>
          </b-tab>

          <!-- STRING TO BYTES32 -->
          <b-tab :title="options.bytes32ToString">
            <b-card-text>
              <b-row align-v="center">
                <b-col md="5">
                  <label class="float-left text-uppercase font-weight-bold">Bytes32</label>
                  <b-form-textarea
                    id="textarea"
                    v-model="bytes32"
                    placeholder="bytes32..."
                    rows="6"
                    max-rows="6"
                    @input="setString"
                  ></b-form-textarea>
                </b-col>

                <b-col md="2">
                  <b-button
                    class="m-3"
                    variant="success"
                    @click="saveRecord('string', string, 'bytes32', bytes32)"
                  >Save</b-button>
                </b-col>

                <b-col md="5">
                  <label class="float-left text-uppercase font-weight-bold">String</label>
                  <b-form-textarea
                    id="textarea"
                    v-model="string"
                    placeholder="string..."
                    rows="6"
                    max-rows="6"
                    @input="setBytes32"
                  ></b-form-textarea>
                </b-col>
              </b-row>
            </b-card-text>
          </b-tab>
        </b-tabs>
      </b-card>
      <b-card class="mt-3">
        <b-row>
          <b-col cols="12">
            <label class="float-left text-uppercase font-weight-bold">Saved records</label>
          </b-col>
        </b-row>
        <b-row>
          <b-col>
            <div>
              <b-table striped hover :items="savedRecords" :fields="fields">
                <template slot="inputData" slot-scope="data">
                  <pre><code>{{ data.value }}</code> </pre>
                </template>
                <template slot="outputData" slot-scope="data">
                  <pre><code>{{ data.value }}</code> </pre>
                </template>
                <template slot="inputType" slot-scope="data">
                  <b-badge pill variant="dark">{{data.value}}</b-badge>
                </template>
                <template slot="outputType" slot-scope="data">
                  <b-badge pill variant="dark">{{data.value}}</b-badge>
                </template>
              </b-table>
            </div>
          </b-col>
        </b-row>
      </b-card>
    </div>
  </b-container>
</template>

<script lang="ts">
import Vue from "vue";
import { ethers } from "ethers";
type Record = {
  inputType: String;
  inputData: String;
  outputType: String;
  outputData: String;
};

export default Vue.extend({
  name: "home",
  components: {},
  data() {
    return {
      options: {
        bytes32ToString: "Bytes32 to String",
        stringToBytes32: "String to Bytes32"
      },
      string: "",
      bytes32: "",
      savedRecords: [],
      fields: [
        {
          key: "inputType",
          sortable: true
        },
        {
          key: "inputData",
          sortable: false
        },
        {
          key: "outputType",
          sortable: true
        },
        {
          key: "outputData",
          sortable: false
        }
      ]
    };
  },
  created() {
    this.loadRecordsToState();
  },
  methods: {
    makeToast(
      message: string,
      variant: string = "primary",
      title: string = "Notis"
    ) {
      this.$bvToast.toast(message, {
        title: title,
        variant: variant,
        solid: true,
        toaster: "b-toaster-bottom-right"
      });
    },
    setBytes32(event: string) {
      try {
        this.bytes32 = ethers.utils.formatBytes32String(event);
      } catch (error) {
        this.makeToast(error.message, "danger");
      }
    },
    setString(event: string) {
      try {
        this.string = ethers.utils.parseBytes32String(event);
      } catch (error) {
        this.makeToast(error.message, "danger");
      }
    },
    saveRecord(
      inputType: String,
      inputData: String,
      outputType: String,
      outputData: String
    ) {
      let saveRecordsArray = this.findRecords();
      const newSaveRecordsArray = [
        ...saveRecordsArray,
        {
          inputType: inputType,
          inputData: inputData,
          outputType: outputType,
          outputData: outputData
        }
      ];
      const newSavedRecordsString = JSON.stringify(newSaveRecordsArray);
      try {
        localStorage.setItem("savedRecords", newSavedRecordsString);
      } catch (error) {
        this.makeToast(error.message, "danger");
      }
      this.loadRecordsToState();
    },
    findRecords() {
      const savedRecordsString = localStorage.getItem("savedRecords");
      let saveRecordsArray = [];
      if (savedRecordsString === null) {
        this.makeToast(
          "Did not find any saved records, so creating a new localStorage data point.",
          "warning"
        );
        saveRecordsArray = [];
      } else {
        saveRecordsArray = JSON.parse(savedRecordsString);
      }
      return saveRecordsArray;
    },
    loadRecordsToState() {
      this.savedRecords = this.findRecords();
    }
  }
});
</script>
