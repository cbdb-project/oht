<template>
  <v-container fluid style="width:600px">
    <v-row>
      <v-col class="text-center">
          <h1>{{single_search_result_items.section_text}}</h1>
      </v-col>
    </v-row>
    <v-row> 
      <v-col><v-card-text>
        <v-combobox
          type="select"
          v-model= "person_name"
          label="Biographical Subject"
          outlined
          dense
        ></v-combobox>
      </v-card-text></v-col>
      <v-col><v-card-text>
          <v-text-field
            v-model="person_id"
            type="message4"
            label="Biographical Subject ID"
            outlined
            clearable
            dense
          ></v-text-field>
      </v-card-text></v-col>
    </v-row>
    <v-row> 
      <v-col><v-card-text>
          <v-select
          v-model="data_type"
          :items="data_type_list"
          label="Data Type"
          dense
          solo
          @change="update_data_type($event)"
        ></v-select>
      </v-card-text></v-col>
      <v-col><v-card-text>
          <v-select
          v-model="data_subtype"
          :items="data_subtype_list"
          label="Data Subtype"
          dense
          solo
          @click="get_data_subtype_list_for_select"
          @change="update_data_subtype($event)"
        ></v-select>
      </v-card-text></v-col>
    </v-row>
    <v-row> 
      <v-col><v-card-text>
          <v-text-field
            v-model="content"
            type="message1"
            label="Content"
            outlined
            clearable
          ></v-text-field>
      </v-card-text></v-col>
    </v-row>
    <v-row> 
      <v-col><v-card-text>
          <v-text-field
            v-model="content_refined"
            type="message2"
            label="Refined Content"
            outlined
            clearable
          ></v-text-field>
      </v-card-text></v-col>
    </v-row>
    <v-row class="text-center"> 
      <v-col><v-card-text>
    <v-btn
      class="mr-4"
      color="error"
      v-on:click="test"
    >
      Close
    </v-btn>
    <v-btn
      class="mr-4"
      color="primary"
      v-on:click="submit_data"
    >
      Save
    </v-btn>


      </v-card-text></v-col>
    </v-row>
    <v-row>
      <v-col class="text-center">
          <span style="color:grey">Context</span><v-divider></v-divider>
          <h2>{{single_search_result_items.section_context}}</h2>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import axios from 'axios'
import global_var from './global_var.vue'
export default {
  data () {
    return {
      tag_id: "",
      single_section_id: "",
      section_text: "",
      section_context: "",
      content: "",
      content_refined: "",
      single_search_result_items: [],
      biog_subjects: [],
      person_id: "",
      person_name: "",
      data_type: "",
      data_type_id: "",
      data_subtype: "",
      data_subtype_id: "",
      data_type_list: [],
      data_subtype_list: [],
    }
  },
  methods: {
    load_section: function(single_section_id){
      let query_url = global_var.url + "/official_hist_tagging/query_type_list.php";
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.data_type_list = response.data;
      }).catch(function (error) {
        console.log(error);
      });
      this.search_term = single_section_id;
      query_url = global_var.url + "/official_hist_tagging/query_tags.php?tag_id=" + single_section_id;
      _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.single_search_result_items = response.data;
        _this.tag_id = response.data.tag_id;
        _this.single_section_id = response.data.section_id;
        _this.section_text = response.data.section_text;
        _this.content = response.data.content
        _this.content_refined = response.data.content_refined;
        _this.person_id = response.data.biog_subject_id;
        _this.person_name = response.data.biog_subject_name
        _this.data_type = response.data.type + "-" + response.data.type_chn;
        _this.data_type_id = response.data.type;
        _this.data_subtype = response.data.subtype + "-" +response.data.subtype_chn;
        _this.data_subtype_id = response.data.subtype;
        _this.data_subtype_list.push(_this.data_subtype);
      }).catch(function (error) {
        console.log(error);
      });
    },
    submit_data: function(){
      let query_url = global_var.url + `/official_hist_tagging/update_main.php?tag_id=${this.tag_id}&biog_subject_id=${this.person_id}&biog_subject_name=${this.person_name}&type=${this.data_type_id}&subtype=${this.data_subtype_id}&content=${this.content}&content_refined=${this.content_refined}`;
      console.log(query_url);
      axios.get(query_url)
      .then(function (response) {
        console.log(response.data);
        alert("Submitted");
      }).catch(function (error) {
        console.log(error);
      });
    },
    update_data_type: function(e){
      this.data_type_id = e.split("-")[0];
      this.data_subtype = "";
      this.data_subtype_list = [];
    },
    update_data_subtype: function(e){
      this.data_subtype_id = e.split("-")[0];
      this.data_subtype = e.split("-")[1];
      console.log(this.data_subtype_id);
      console.log(this.data_subtype);
    },
    get_data_subtype_list_for_select: function(){
      let query_url = global_var.url + "/official_hist_tagging/query_subtype_list_by_typeid.php?typeid="+this.data_type_id;
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.data_subtype_list = response.data;
      }).catch(function (error) {
        console.log(error);
      });
    },
    test: function($e){
      console.log($e);
    },
  },
  beforeMount(){
    let section_id = this.$route.query.tag_id;
    this.load_section(section_id);
 }
}
</script>