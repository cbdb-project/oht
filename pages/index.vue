<template>
  <v-row>
    <v-col align="center">
    <v-row>
    <v-col align="left" justify="left">
      <v-card>
        <v-card-title class="headline">
      <div class="text-center">
        <v-text-field
          label="Chapter ID"
           v-model="chapter_id"
           v-on:keyup.enter="call_search_api"
        ></v-text-field>

        <v-btn
          class="mr-4"
          v-on:click="call_search_api"
          id="sbt"
        >
          Search
        </v-btn>
        <v-btn
          class="mr-4"
          v-on:click="clear_search_box"
        >
          Clear
        </v-btn>
      </div><br />
        </v-card-title>
        <h4><p style="margin-left:15px;margin-right:15px">{{search_result_items_title}}</p></h4>
        <v-card-text>
          <v-list>
            <v-list-item v-for="section in search_result_items.sections" :key="section.section_id">
              <div>
              <v-card-title class="headline">{{section.section_text}}</v-card-title>
              <v-card-text><v-btn
                color="green"
                elevation="2"
                small
                v-if="section.section_text"
                @click="add_new_tag(section.section_id, section.section_text, section.biog_subject_id, section.biog_subject_name)"
              >
              Add</v-btn></v-card-text>
              <v-list-item v-for="tag_record in section.tag_records" :key="tag_record.tag_id">
              <v-card-text class="headline">
              <v-btn
              large
              elevation="2"
              @click="load_section(tag_record.tag_id)"
              >
              ( {{tag_record.biog_subject_name}}-<font color="blue">{{tag_record.type_chn}}</font>-{{tag_record.subtype_chn}}-<font color="blue">{{tag_record.content_refined}}</font>)
              </v-btn>
              </v-card-text>
              <v-btn
              color="error"
              elevation="2"
              small
              @click="remove_tag(section.section_id, tag_record.tag_id)"
              >
              Del</v-btn>
              </v-list-item><br /><br /><v-divider></v-divider>
              </div>
            </v-list-item>
          </v-list>
        </v-card-text>

      </v-card>
    </v-col>
 
 </v-row>
   <v-row justify="center" align="center"><v-col>
  <!-- <v-container fluid style="width:600px"> -->
        <v-overlay
          :absolute="absolute"
          :opacity="opacity"
          :value="overlay"
        >
    <v-row>
      <v-col class="text-center">
          <h1 @mouseup="put_selection_text_to_content">{{section_text}}</h1>
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
          :items="person_name_list"
          @input.native="person_name=$event.srcElement.value"
          @click="load_subjects_in_section"
          @change="select_biog_subject"
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
            v-on:keyup.enter="update_person_name_by_cbdb_id"
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
    <v-row> 
      <v-col><v-card-text>
        <v-combobox
          type="select"
          v-model= "content_person_name"
          label="Content Person Name"
          outlined
          dense
          :items="person_name_list"
          @input.native="content_person=$event.srcElement.value"
          @click="load_subjects_in_section"
          @change="select_content_person"
        ></v-combobox>
      </v-card-text></v-col>
      <v-col><v-card-text>
          <v-text-field
            v-model="content_personid"
            type="message4"
            label="Content Person ID"
            outlined
            clearable
            dense
            v-on:keyup.enter="update_content_name_by_cbdb_id"
          ></v-text-field>
      </v-card-text></v-col>
    </v-row>
    <v-row class="text-center"> 
      <v-col><v-card-text>
    <v-btn
      class="mr-4"
      color="error"
      @click="overlay = false"
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
    <!-- <v-row>
      <v-col class="text-center">
          <span style="color:grey">Context</span><v-divider></v-divider>
          <h2>{{single_search_result_items.section_context}}</h2>
      </v-col>
    </v-row> -->
  </v-overlay>
    <!-- </v-container> -->
    </v-col></v-row>
 </v-col></v-row>
</template>

<script>
import axios from 'axios'
import global_var from './global_var.vue'
export default {
  data () {
    return {
      chapter_id:"",
      section_id: "",
      search_result_items: [],
      search_result_items_title: "",
      tag_counter: 0,
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
      person_name_list: [],
      data_type: "",
      data_type_id: "",
      data_subtype: "",
      data_subtype_id: "",
      data_type_list: [],
      data_subtype_list: [],
      content_personid: "",
      content_person_name: "",
      content_person: "",
      absolute: false,
      opacity: 0.85,
      overlay: false,
      light: true,
    }
  },
  methods: {
    put_selection_text_to_content: function() {
      let selected_text = "";
      if (window.getSelection) {
          selected_text = window.getSelection().toString();
      } else if (document.selection && document.selection.type != "Control") {
          selected_text = document.selection.createRange().text;
      }
      this.content = selected_text;
      console.log(selected_text);
      navigator.clipboard.writeText(selected_text)
      // navigator.clipboard.write(text).then(function () {
      // }, function (err) {
      //   console.log(err);
      // })
    },
    update_person_name_by_cbdb_id: function(){
      let query_url = global_var.url + "/official_hist_tagging/query_person_name_by_personid.php?person_id=" + this.person_id;
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        // console.log(response.data);
        // console.log(query_url);
        _this.person_name = response.data;
      }).catch(function (error) {
        console.log(error);
      });
    },
    update_content_name_by_cbdb_id: function(){
      let query_url = global_var.url + "/official_hist_tagging/query_person_name_by_personid.php?person_id=" + this.content_personid;
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        // console.log(response.data);
        // console.log(query_url);
        _this.content_person_name = response.data;
      }).catch(function (error) {
        console.log(error);
      });
    },
    select_biog_subject:function(){
      this.person_id = this.person_name.split("-")[0];
      this.person_name = this.person_name.split("-")[1];

    },
    select_content_person:function(){
      this.content_personid = this.content_person_name.split("-")[0];
      this.content_person_name = this.content_person_name.split("-")[1];
      console.log( this.content_person.split("-"));

    },
    load_subjects_in_section:function(){
      let query_url = global_var.url + "/official_hist_tagging/query_subjects_in_section.php?chapter_id=" + this.chapter_id;
      console.log(query_url)
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        // console.log(response.data);
        // console.log(query_url);
        _this.person_name_list = response.data;
      }).catch(function (error) {
        console.log(error);
      });
    },
    remove_tag:function(section_id, tag_id){
      let confirm_delete = confirm("Confirm to delete this record");
      if (confirm_delete == true){
        let query_url = global_var.url + "/official_hist_tagging/delete_main.php?tag_id="+tag_id;
        let _this = this;
        axios.get(query_url)
        .then(function (response) {
          // console.log(response.data);
          // console.log(query_url);
          // _this.person_name = response.data;
          // section_id-=1
          // let tag_index = 0
          // for (const [tag_records_index, tag_records_value_list] of Object.entries(_this.search_result_items.sections[section_id].tag_records)) {
          //   if(tag_id == tag_records_value_list.tag_id){
          //     tag_index = tag_records_index;
          //     break
          //   }
          // }
          // _this.$delete(_this.search_result_items.sections[section_id].tag_records, tag_index);
          document.getElementById("sbt").click(); 
          // alert("Done");
          }).catch(function (error) {
            console.log(error);
          });
        }
    },
    add_new_tag: function(current_section_id, current_section_text, current_biog_subject_id, current_biog_subject_name){
      this.overlay = "!overlay";
      this.tag_id = "new";
      this.single_section_id = current_section_id;
      this.section_text = current_section_text;
      this.content = "";
      this.content_refined = "";
      this.content_personid = "";
      this.content_person_name = "";
      this.person_id = current_biog_subject_id;
      this.person_name = current_biog_subject_name;
      let query_url = global_var.url + "/official_hist_tagging/query_type_list.php";
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.data_type_list = response.data;
      }).catch(function (error) {
        console.log(error);
      });
      _this.data_type = "1-基本信息";
      _this.data_type_id = 1;
      _this.data_subtype = "1-姓名（[姓] [名]）";
      _this.data_subtype_id = 1;
      _this.data_subtype_list = ["1-姓名（[姓] [名]）"];
    },
    add_data: function(){
      if (this.content_person_name == undefined){
        this.content_person_name = ""
      }
      if (this.content_personid == undefined){
        this.content_personid = ""
      }
      if (this.content_person_name == undefined){
        this.content_person_name = ""
      }
      let query_url = global_var.url + `/official_hist_tagging/add_main.php?biog_subject_id=${this.person_id}&biog_subject_name=${this.person_name}&type=${this.data_type_id}&subtype=${this.data_subtype_id}&content=${this.content}&content_refined=${this.content_refined}&section_id=${this.single_section_id}&content_personid=${this.content_personid}&content_person_name=${this.content_person_name}`;
      // console.log(query_url);
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        console.log(response.data);
        document.getElementById("sbt").click(); 
      }).catch(function (error) {
        console.log(error);
      });
    },
    clear_search_box: function () {
      this.chapter_id = "";
    },
    call_search_api: function(){
      let query_url = global_var.url+"/official_hist_tagging/query_main.php?chapter_id=" + this.chapter_id;
      let _this = this;
      // _this.search_result_items = [{word:"Searching...", content: "", id: 0}]
      axios.get(query_url)
      .then(function (response) {
        _this.search_result_items = response.data;
        _this.search_result_items_title = response.data.book_name + "/" + response.data.chapter_name;
      }).catch(function (error) {
        console.log(error);
      });
    },
    load_section: function(single_tag_id){
      this.overlay = "!overlay";
      let query_url = global_var.url + "/official_hist_tagging/query_type_list.php";
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.data_type_list = response.data;
      }).catch(function (error) {
        console.log(error);
      });
      query_url = global_var.url + "/official_hist_tagging/query_tags.php?tag_id=" + single_tag_id;
      _this = this;
      axios.get(query_url)
      .then(function (response) {
        _this.single_search_result_items = response.data;
        _this.tag_id = response.data.tag_id;
        _this.single_section_id = response.data.section_id;
        _this.section_text = response.data.section_text;
        _this.content = response.data.content;
        _this.content_refined = response.data.content_refined;
        console.log()
        _this.person_id = response.data.biog_subject_id;
        _this.person_name = response.data.biog_subject_name;
        _this.data_type = response.data.type + "-" + response.data.type_chn;
        _this.data_type_id = response.data.type;
        _this.data_subtype = response.data.subtype + "-" + response.data.subtype_chn;
        _this.data_subtype_id = response.data.subtype;
        _this.data_subtype_list.push(_this.data_subtype);
        _this.content_personid = response.data.content_personid;
        _this.content_person_name = response.data.content_person_name;
        _this.content_person = response.data.content_personid + "-" + response.data.content_person_name
      }).catch(function (error) {
        console.log(error);
      });
    },
    submit_data: function(){
      if (this.content_person_name == undefined){
        this.content_person_name = ""
      }
      if (this.content_personid == undefined){
        this.content_personid = ""
      }
      if (this.content_refined == "" || this.content_refined == "null"){
        this.content_refined = this.content;
      }
      if(this.tag_id == "new"){
        this.add_data();
        this.overlay = false;
      }else{
        let query_url = global_var.url + `/official_hist_tagging/update_main.php?tag_id=${this.tag_id}&biog_subject_id=${this.person_id}&biog_subject_name=${this.person_name}&type=${this.data_type_id}&subtype=${this.data_subtype_id}&content=${this.content}&content_refined=${this.content_refined}&content_personid=${this.content_personid}&content_person_name=${this.content_person_name}`;
        console.log(query_url);
        let _this = this;
        axios.get(query_url)
        .then(function (response) {
          console.log(response.data);
          let current_section_id = parseInt(_this.single_section_id)-1;
          let tag_index = 0;
          for (const [tag_records_index, tag_records_value_list] of Object.entries(_this.search_result_items.sections[current_section_id].tag_records)) {
            if(_this.tag_id == tag_records_value_list.tag_id){
              tag_index = tag_records_index;
              break
            }
          }
          // console.log("old_section_id:"+_this.single_section_id);
          // console.log("current_section_id:"+current_section_id);
          // console.log("tag_index:"+tag_index);
          // console.log(_this.search_result_items.sections[current_section_id].tag_records[tag_index]);
          console.log("start:")
          console.log("original biog_subject_id:")
          console.log(_this.search_result_items.sections[current_section_id])
          console.log("person_id:"+_this.person_id)
          console.log("tag_index:"+tag_index)
          console.log("tag_index:"+current_section_id)
          console.log("end")
          document.getElementById("sbt").click(); 
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].biog_subject_id = _this.person_id;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].biog_subject_name = _this.person_name;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].content = _this.content;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].subtype = _this.data_subtype_id;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].subtype_chn = _this.data_subtype;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].type = _this.data_type_id;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].subtype_chn = _this.data_type;
          // _this.search_result_items.sections[current_section_id].tag_records[tag_index].tag_id = _this.tag_id;    
          _this.overlay = false;
        }).catch(function (error) {
          console.log(error);
        });
      }
      
    },
    update_data_type: function(e){
      this.data_type_id = e.split("-")[0];
      this.data_subtype = "";
      this.get_data_subtype_list_for_select();
      // this.data_subtype_list = [];
    },
    update_data_subtype: function(e){
      this.data_subtype_id = e.split("-")[0];
      // this.data_subtype = e.split("-")[1];
      this.data_subtype = e;
    },
    get_data_subtype_list_for_select: function(){
      let query_url = global_var.url + "/official_hist_tagging/query_subtype_list_by_typeid.php?typeid="+this.data_type_id;
      let _this = this;
      axios.get(query_url)
      .then(function (response) {
        console.log(query_url);
        _this.data_subtype_list = response.data;
        _this.data_subtype = _this.data_subtype_list[0];
        _this.data_subtype_id = _this.data_subtype_list[0].split("-")[0];
      }).catch(function (error) {
        console.log(error);
      });
    },
    test: function($e){
      console.log($e);
    },

  },
  beforeMount(){
    if (typeof this.$route.query.id !== 'undefined') {
      let chapter_id = this.$route.query.id;
      this.chapter_id = chapter_id;
      this.call_search_api();
    }
  }

}

</script>
