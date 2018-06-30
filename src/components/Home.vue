<template>
  <v-container fluid>
    <v-slide-y-transition mode="out-in">
      <v-layout row wrap>
        <v-flex xs12 sm12 style="text-align: center;">
            <v-progress-circular v-if="operation" indeterminate color="green">File Uploading...</v-progress-circular>
            <v-btn raised dark class="blue lighten-1" @click="onPickFile" :disabled="operation">Upload Image</v-btn>
            <input type="file" 
            style="display:none" 
            ref="fileInput"
            accept="text/plain"
            @change="onFilePicked"
            />
          </v-flex>
          <v-flex xs12 sm12>
            <v-layout row wrap>
              <v-flex xs12 sm3 v-for="item in fileList" :key="item.name" style="margin-bottom: 2px;">
                <v-card>
                  <v-card-title primary-title>{{item.name}}
                   </v-card-title>
                </v-card>
              </v-flex>
            </v-layout>
          </v-flex>
      </v-layout>
    </v-slide-y-transition>
  </v-container>
</template>

<script>
import firebase from "firebase";
export default {
  data() {
    return {
      testFile: "",
      operation: false,
      fileList: ""
    };
  },
  mounted() {
    firebase
      .database()
      .ref("files")
      .on("value", snapshot => {
        this.fileList = [];
        let dblist = snapshot.val();
        for (let key in dblist) {
          this.fileList.push(dblist[key]);
        }
      });
  },
  methods: {
    onPickFile() {
      this.$refs.fileInput.click();
    },
    onFilePicked(event) {
      let files = event.target.files;
      let fileName = files[0].name;
      if (fileName.lastIndexOf(".") <= 0 || fileName.indexOf(".txt") < 0) {
        return alert("Please add valid text file");
      }
      this.testFile = files[0];
      this.uploadOnServer();
      // console.log(files[0].split(".txt"));
    },
    uploadOnServer() {
      this.operation = true;
      const filename = this.testFile.name;
      const file = filename.split(".txt")[0];
      const ext = filename.slice(filename.lastIndexOf("."));
      firebase
        .storage()
        .ref("files/" + file + "" + ext)
        .put(this.testFile)
        .then(fileData => {
          firebase
            .database()
            .ref("files")
            .push({ name: fileData.metadata.name })
            .then(res => {
              this.operation = false;
            })
            .catch(error => {
              console.log(error);
              this.operation = false;
            });
        })
        .catch(error => {
          this.operation = false;
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
