<template>
  <div>
    <h1>Manage Rooms</h1>
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
      <div class="panel-heading">Add a Room</div>
      <div class="panel-body">
        <b-form id="Add_Room_Form">
          <b-col cols="12" md="6" lg="2">
            <b-form-group
              class="roomnumber"
              label="Room Number"
              label-for="roomNumber">
              <b-form-input
                type="text"
                v-model="room.room_number"
                id="roomNumber"
                required></b-form-input>
            </b-form-group>
          </b-col>

          <!--  Room Name -->
          <b-col cols="12" md="6" lg="4">
            <b-form-group
              class="roomname"
              label="Room Name"
              label-for="roomName">
              <b-form-input
                type="text"
                v-model="room.room_name"
                id="roomName"
                required></b-form-input>
            </b-form-group>
          </b-col>

          <!--  Room Type -->
          <b-col cols="12" md="6" lg="4">
            <b-form-group
              class="roomtype"
              label="Room Type"
              label-for="roomType">
              <b-form-input
                type="text"
                v-model="room.room_type"
                id="roomType"
                required></b-form-input>
            </b-form-group>
          </b-col>

          <!-- Room Capacity -->
          <b-col cols="12" md="6" lg="2">
            <b-form-group
              class="roomcapacity"
              label="Room Capacity"
              label-for="roomCapacity">
              <b-form-input
                type="number"
                v-model="room.room_capacity"
                id="roomCapacity"
                required></b-form-input>
            </b-form-group>
          </b-col>

            </b-col>
          </b-form-row>
          <b-form-row>
            <b-col>
              <b-button variant="danger" @click="toggleForm">
                Cancel
              </b-button>
            </b-col>
            <b-col class="d-flex justify-content-end">
              <b-button variant="primary" id="Add_Room_Btn" @click="addRoom">
                Add
              </b-button>
            </b-col>
          </b-form-row>
        </b-form>
      </div>
    </div>
    <!--Form end  -->

    <b-button variant="success" size="sm" @click="toggleForm" class="toggleFormBtn" v-if="!showForm">
      Add New Room
    </b-button>

    <ag-grid-vue class="ag-theme-material"
      :columnDefs="columnDefs"
      :rowData="rowData"
      :animateRows="true"
      :pagination="true"
      :paginationPageSize="10"
      :gridOptions="gridOptions">
    </ag-grid-vue>

    <b-modal id="editRoomModal" ref="editRoomModal" title="Edit Room" size="xl">
      <b-form-row>
      <!-- Room Number -->
      <b-col cols="12" md="6" lg="2">
        <b-form-group
          class="roomnumber"
          label="Room Number"
          label-for="roomNumber">
          <b-form-input
            type="text"
            v-model="room.room_number"
            id="roomNumber"
            required></b-form-input>
        </b-form-group>
      </b-col>

      <!--  Room Name -->
      <b-col cols="12" md="6" lg="4">
        <b-form-group
          class="roomname"
          label="Room Name"
          label-for="roomName">
          <b-form-input
            type="text"
            v-model="room.room_name"
            id="roomName"
            required></b-form-input>
        </b-form-group>
      </b-col>

      <!--  Room Type -->
      <b-col cols="12" md="6" lg="4">
        <b-form-group
          class="roomtype"
          label="Room Type"
          label-for="roomType">
          <b-form-input
            type="text"
            v-model="room.room_type"
            id="roomType"
            required></b-form-input>
        </b-form-group>
      </b-col>

      <!-- Room Capacity -->
      <b-col cols="12" md="6" lg="2">
        <b-form-group
          class="roomcapacity"
          label="Room Capacity"
          label-for="roomCapacity">
          <b-form-input
            type="number"
            v-model="room.room_capacity"
            id="roomCapacity"
            required></b-form-input>
        </b-form-group>
      </b-col>
      </b-form-row>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('editRoomModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="updateRoom()">
          Update
        </b-button>
      </template>
    </b-modal>

    <b-modal id="deleteRoomModal" ref="deleteRoomModal" title="Delete Room" size="lg">
      <p>Are you sure you want to delete {{ this.room.room_number }}?</p>

      <template v-slot:modal-footer="{ cancel, ok }">
        <!-- Emulate built in modal footer ok and cancel button actions -->
        <b-button size="sm" variant="danger" @click="$bvModal.hide('deleteRoomModal')">
          Cancel
        </b-button>
        <b-button size="sm" variant="success" @click="deleteRoom()">
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
  import RoomsActionButtons from "./ActionButtons/RoomsActionButtons.vue";
  export default{

    name: 'RoomsManager',
    components: {
      AgGridVue,
      RoomsActionButtons
    },
    data() {
      return {
        columnDefs: null,
        rowData: null,
        gridOptions: null,
        id: null,
        room: {
          room_number: null,
          room_name: null,
          room_type: null,
          room_capacity: 0,
          active: 1
        },
        options: [
          {value: 0, text: 'Inactive'},
          {value: 1, text: 'Active'}
        ],
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
          {headerName: 'Room Number', field: 'room_number', sortable: true, filter: true, width: 130,},
          {headerName: 'Room Name', field: 'room_name', sortable: true, filter: true, resizable: true},
          {headerName: 'Room Type', field: 'room_type', sortable: true, filter: true, width: 150},
          {headerName: 'Room Capacity Units', field: 'room_capacity', sortable: true, filter: true, width: 180,},
          {headerName: 'Status', field: 'active', width: 120,
            cellRenderer: (data) => {
              if(data.value === 1){
                return 'Active';
              } else {
                return 'Inactive';
              }
            }},
          {headerName: 'Actions', field: 'id', cellRendererFramework: 'RoomsActionButtons'}
      ];

    },
    mounted () {
      this.getRooms();
    },
    methods:{
      getRooms: function(){
        Axios
          .get('http://localhost/api/v1/rooms', {
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
      addRoom: function(){
        this.errors = [];
        Axios
          .post('http://localhost/api/v1/rooms', this.room, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getRooms();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.room = {
              room_number: null,
              room_name: null,
              room_type: null,
              room_capacity: 0,
              active: 1
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
      updateRoom: function(){
        this.errors = [];
        Axios
        .put('http://localhost/api/v1/rooms/' + this.id, this.room, {
          headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
        })
        .then(response => {
          this.getRooms();
          this.alertMessage = response.data.message;
          this.dismissSuccessCountDown = this.dismissSecs;
          // clear subject data
          this.room = {
            room_number: null,
            room_name: null,
            room_type: null,
            room_capacity: 0,
            active: 1
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
        this.$refs['editRoomModal'].hide();
      },
      deleteRoom: function(){
        this.errors = [];
        Axios
          .delete('http://localhost/api/v1/rooms/' + this.id, {
            headers: {'Authorization': 'Bearer ' + this.$store.getters.getToken}
          })
          .then(response => {
            this.getRooms();
            this.alertMessage = response.data.message;
            this.dismissSuccessCountDown = this.dismissSecs;
            // clear room data
            this.room = {
              room_number: null,
              room_name: null,
              room_type: null,
              room_capacity: 0,
              active: 1
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
        this.$refs['deleteRoomModal'].hide();
      }
    }
  }
</script>
