<template>
  <div>
    <h1>Manage Academic Year</h1>
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
      <div class="panel-heading">Add a Academic Year</div>
      <div class="panel-body">
        <b-form id="Add_AY_Form">
          <b-col cols="12" md="6" lg="2">
            <b-form-group
              class="academicyear"
              label="Academic Year"
              label-for="academicYear">
              <b-form-input
                type="text"
                v-model="AcademicYears.academic_year"
                id="academicYear"
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
              <b-button variant="primary" id="Add_AY_Btn" @click="addAY">
                Add
              </b-button>
            </b-col>
          </b-form-row>
        </b-form>
      </div>
    </div>
    <!--Form end  -->

    <b-button variant="success" size="sm" @click="toggleForm" class="toggleFormBtn" v-if="!showForm">
      Add New Academic Year
    </b-button>

    <ag-grid-vue class="ag-theme-material"
      :columnDefs="columnDefs"
      :rowData="rowData"
      :animateRows="true"
      :pagination="true"
      :paginationPageSize="10"
      :gridOptions="gridOptions">
    </ag-grid-vue>

    <b-modal id="editAYModal" ref="editAYModal" title="Edit Academic Year" size="xl">
      <b-form-row>
      <!-- Room Number -->
      <b-col cols="12" md="6" lg="2">
        <b-form-group
          class="academicyear"
          label="Academic Year"
          label-for="academicYear">
          <b-form-input
            type="text"
            v-model="AcademicYears.academic_year"
            id="academicYear"
            required></b-form-input>
        </b-form-group>
      </b-col>
      </b-form-row>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('editAYModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="updateAY">
          Update
        </b-button>
      </template>
    </b-modal>

    <b-modal id="deleteAYModal" ref="deleteAYModal" title="Delete AY" size="lg">
      <p>Are you sure you want to delete {{ this.AcademicYears.academic_year }}?</p>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('deleteAYModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="deleteAY">
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
  import AcademicYearActionButtons from "./ActionButtons/AcademicYearActionButtons.vue";
  export default{
    name: 'AcademicYearManager',
    components: {
      AgGridVue,
      AcademicYearActionButtons
    },
    data() {
      return {
        columnDefs: null,
        rowData: null,
        gridOptions: null,
        id: null,
        AcademicYears: {
          academic_year: null
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
          {headerName: 'Academic Year', field: 'academic_year', sortable: true, filter: true, width: 130,},
          {headerName: 'Actions', field: 'id', cellRendererFramework: 'AcademicYearActionButtons'}
      ];

    },
    mounted () {
      this.getAcademicYear();
    },
    methods:{
      getAcademicYear: function(){
        Axios
          .get('http://localhost/api/v1/academic_years', {
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
      addAY: function(){
        this.errors = [];
        Axios
          .post('http://localhost/api/v1/academic_years', this.AcademicYears, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getAcademicYear();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.AcademicYears = {
              academic_year: null,
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
      updateAY: function(){
        this.errors = [];
        Axios
        .put('http://localhost/api/v1/academic_years/' + this.id, this.AcademicYears, {
          headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
        })
        .then(response => {
          this.getAcademicYear();
          this.alertMessage = response.data.message;
          this.dismissSuccessCountDown = this.dismissSecs;
          // clear subject data
          this.AcademicYears = {
            academic_year: null,
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
        this.$refs['editCourseModal'].hide();
      },
      deleteAY: function(){
        this.errors = [];
        Axios
          .delete('http://localhost/api/v1/academic_years/' + this.id, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getAcademicYear();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.AcademicYears = {
              academic_year: null,
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
        this.$refs['deleteAYModal'].hide();
      }
    }
  }
</script>
