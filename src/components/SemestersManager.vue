<template>
  <div>
    <h1>Manage Semesters</h1>
    <!--form Start  -->
    <b-alert variant="success"
      :show="dismissSuccessCountDown"
      @dismissed="dismissSuccessCountDown=0"
      dismissible fade>
        {{alertMessage}}
    </b-alert>
    <b-alert variant="danger"
      :show="dismissErrorCountDown"
      @dismissed="dismissErrorCountDown=0"
      dismissible fade>
        <p>{{alertMessage}}</p>
        <ul>
          <li v-for="error in errors">{{ error }}</li>
        </ul>
    </b-alert>
    <div class="panel panel-primary recordMaintenanceForm" v-if="showForm">
      <div class="panel-heading">Add a Semester</div>
      <div class="panel-body">
        <b-form id="Add_Semester_Form">
          <b-col cols="12" md="6" lg="2">
            <b-form-group
              class="semester"
              label="Semester"
              label-for="Semester">
              <b-form-input
                type="text"
                v-model="semesters.semester"
                id="Semester"
                required></b-form-input>
            </b-form-group>
          </b-col>

          <b-form-row>
            <b-col>
              <b-button variant="danger" @click="toggleForm">
                Cancel
              </b-button>
            </b-col>
            <b-col class="d-flex justify-content-end">
              <b-button variant="primary" id="Add_Semester_Btn" @click="addSemeter">
                Add
              </b-button>
            </b-col>
          </b-form-row>
        </b-form>
      </div>
    </div>
    <!--Form end  -->

    <b-button variant="success" size="sm" @click="toggleForm" class="toggleFormBtn" v-if="!showForm">
      Add New Semster
    </b-button>

    <ag-grid-vue class="ag-theme-material"
      :columnDefs="columnDefs"
      :rowData="rowData"
      :animateRows="true"
      :pagination="true"
      :paginationPageSize="10"
      :gridOptions="gridOptions">
    </ag-grid-vue>

    <b-modal id="editSemesterModal" ref="editSemesterModal" title="Edit Semester" size="xl">
      <b-form-row>
      <!-- Semester  -->
      <b-col cols="12" md="6" lg="2">
        <b-form-group
          class="semester"
          label="Semester"
          label-for="Semester">
          <b-form-input
            type="text"
            v-model="semesters.semester"
            id="roomNumber"
            required></b-form-input>
        </b-form-group>
      </b-col>
      </b-form-row>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('editSemesterModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="updateSemester()">
          Update
        </b-button>
      </template>
    </b-modal>

    <b-modal id="deleteSemesterModal" ref="deleteSemesterModal" title="Delete Semester" size="lg">
      <p>Are you sure you want to delete {{ this.semesters.semester }}?</p>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('deleteSemesterModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="deleteSemester()">
          Delete
        </b-button>
      </template>
    </b-modal>
  </div>
</template>   

<script>
  import Axios from "axios";
  import {AgGridVue} from "ag-grid-vue";
  import '../../node_modules/ag-grid-community/dist/styles/ag-grid.css';
  import '../../node_modules/ag-grid-community/dist/styles/ag-theme-material.css';
  import SemestersActionButtons from "./ActionButtons/SemestersActionButtons.vue";
  export default{
    name: 'SemestersManager',
    components: {
      AgGridVue,
      SemestersActionButtons
    },
    data() {
      return {
        columnDefs: null,
        rowData: null,
        gridOptions: null,
        id: null,
        semesters:{
          semester:null
        },
        showForm: false,
        alertMessage: "",
        errors: [],
        dismissSecs: 7,
        dismissSuccessCountDown: 0,
        dismissErrorCountDown: 0,
      }
    },
    beforeMount(){
      this.gridOptions = {
          context: {
              componentParent: this
          }
      };
      this.columnDefs = [
          {headerName: 'ID', field: 'id', sortable: true, filter: true, width: 80},
          {headerName: 'Semester', field: 'semester', sortable: true, filter: true, width: 130,},
          {headerName: 'Actions', field: 'id', cellRendererFramework: 'SemestersActionButtons'}
      ];

    },
    mounted () {
      this.getSemesters();
    },
    methods:{
      getSemesters: function(){
        Axios
          .get('http://localhost/api/v1/semesters', {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            //console.log(response.data.data);
            this.rowData = response.data;
          })
          .catch(error => {
            this.alertMessage = error.response.data.message;
            this.dismissErrorCountDown = this.dismissSecs;
          })
      },
      toggleForm: function(){
        if(this.showForm){
          this.showForm = false;
        } else {
          this.showForm = true;
        }
      },
      addSemeter: function(){
        this.errors = [];
        Axios
          .post('http://localhost/api/v1/semesters', this.semesters, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getSemesters();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.semesters = {
              semester: null
            };
          })
          .catch(error => {
            this.alertMessage = error.response.data.message;
            const values = Object.values(error.response.data.errors);
            for(const val of values){
              for(const err of val){
                this.errors.push(err);
              }
            }
            this.dismissErrorCountDown = this.dismissSecs;
          })
      },
      updateSemester: function(){
        this.errors = [];
        Axios
        .put('http://localhost/api/v1/semesters/' + this.id, this.semesters, {
          headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
        })
        .then(response => {
          this.getSemesters();
          this.alertMessage = response.data.message;
          this.dismissSuccessCountDown = this.dismissSecs;
          // clear subject data
          this.semesters = {
            semester: null
          };
        })
        .catch(error => {
          this.alertMessage = error.response.data.message;
          const values = Object.values(error.response.data.errors);
          for(const val of values){
            for(const err of val){
              this.errors.push(err);
            }
          }
          this.dismissErrorCountDown = this.dismissSecs;
        });
        this.$refs['editSemesterModal'].hide();
      },
      deleteSemester: function(){
        this.errors = [];
        Axios
          .delete('http://localhost/api/v1/semesters/' + this.id, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getSemesters();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.semesters = {
              semester: null
            };
          })
          .catch(error => {
            this.alertMessage = error.response.data.message;
            /*const values = Object.values(error.response.data.errors);
            for(const val of values){
              for(const err of val){
                this.errors.push(err);
              }
            }*/
            this.dismissErrorCountDown = this.dismissSecs;
          });
        this.$refs['deleteSemesterModal'].hide();
      }
    }
  }
</script>
