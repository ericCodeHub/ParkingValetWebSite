<template>
  <div id="dodge">
    <h3>CHECKED IN CARS</h3>
    <b-container fluid>
      <!-- User Interface controls -->
      <b-row>
        <b-col lg="4" class="my-2">
          <b-form-group
            label="Filter"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="filterInput"
            class="mb-2"
            id="filterCarDeets"
          >
            <b-input-group size="sm">
              <b-form-input
                v-model="filter"
                type="search"
                id="filterInput"
                placeholder="Type to Search"
              ></b-form-input>
              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''"
                  >Clear</b-button
                >
              </b-input-group-append>
            </b-input-group>
          </b-form-group></b-col
        >

        <b-col lg="6" class="my-1"> </b-col>
      </b-row>

      <b-table
        show-empty
        small
        stacked="md"
        :items="this.$store.state.checkedInCars"
        :fields="fields"
        :per-page="perPage"
        :filter="filter"
        :filter-included-fields="filterOn"
        :sort-by.sync="sortBy"
        :sort-desc.sync="sortDesc"
        :sort-direction="sortDirection"
        @filtered="onFiltered"
        
      >
      <!--I know now why this is here so commented out
        <template #cell(name)="row">
          where is this?{{ row.value.first }} {{ row.value.last }}
        </template>-->
        <template #cell(amountowed)="row">
          ${{ calcTime(row.item.timeIn) }}
        </template>
        <template #cell(actions)="row" >
          <div v-if="!hideAllButtons">
            <b-button size="sm" @click="row.toggleDetails">
              {{ row.detailsShowing ? "Hide" : "Show" }} Details
            </b-button>
            <b-button size="sm" @click="showParkingSpotForm(row.item.ticketId,false)">
              Park Car
              
            </b-button>
            
            <b-button size="sm" @click="row.toggleDetails">
              Check Out Car
            </b-button>
            <b-button size="sm" @click="row.toggleDetails">
              Request Pickup
            </b-button>
            </div>
            <div>
          <update-spot-id
                        v-if="parkingSpotForm"
                        v-bind:ticketId="ticketId"
                        v-bind:request="request"
                        v-on:unhide-buttons="hideAllButtons=false, ParkingSpotForm = false" 
            />
            </div>
        </template>

        <template #row-details="row">
          <b-card>
            <div id="listStuffModal">
              <div id="displayPatronList">
                <h5>
                  PATRON NAME :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.firstName }} {{ row.item.lastName }}
                  </p>
                </h5>

                <h5>
                  PATRON PHONE NUMBER :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.phoneNumber }}
                  </p>
                </h5>

                <h5>
                  PATRON EMAIL :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.emailAddress }}
                  </p>
                </h5>
              </div>
              <div id="displayCarList">
                <h5>
                  LICENSE PLATE :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.licensePlate }}
                  </p>
                </h5>

                <h5>
                  VEHICLE MAKE :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.vehicleMake }}
                  </p>
                </h5>

                <h5>
                  VEHICLE MODEL :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.vehicleModel }}
                  </p>
                </h5>

                <h5>
                  VEHICLE COLOR :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.vehicleColor }}
                  </p>
                </h5>

                <h5>
                  PARKING SPOT :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.parkingSpotId }}
                  </p>
                  <p>
                    <button @click="showParkingSpotForm(row.item.ticketId, true)">
                      update parking spot
                    </button>
                    <update-spot-id
                      v-if="parkingSpotForm"
                      v-bind:ticketId="ticketId" 
                      v-bind:request="request" 
                      v-on:unhide-buttons="hideAllButtons=false, ParkingSpotForm = false"                                    
                    />
                  </p>
                </h5>
                <h5>
                  TIME IN :
                  <p style="display: inline" class="attributesList">
                    {{ row.item.timeIn.substring(0, 10) }}
                    <b-icon icon="calendar2-date" aria-hidden="true"></b-icon
                    >&nbsp; {{ row.item.timeIn.substring(11, 19) }}
                    <b-icon icon="clock" aria-hidden="true"></b-icon>
                  </p>
                </h5>

                <h5>
                  AMOUNT OWED :
                  <p style="display: inline" class="attributesList">
                    ${{ calcTime(row.item.timeIn) }}
                  </p>
                </h5>
              </div>
            </div>
          </b-card>
        </template>
      </b-table>
    </b-container>
  </div>
</template>

<script>
import ValetService from "@/services/ValetService.js";
import moment from "moment";
import UpdateSpotId from "@/components/UpdateSpotId.vue";

export default {
  name: "list-of-cars",

  components: { UpdateSpotId },
  created() {
    this.UpdateCars();
  },
  data() {
    return {
      fields: [
        { key: "ticketId", sortable: true, class: "text-center" },
        { key: "vehicleMake", label: "make", sortable: true, class: "text-center" },
        { key: "vehicleModel", label: "model", sortable: true, class: "text-center" },
        { key: "licensePlate", label: "plate", sortable: true, class: "text-center" },
        { key: "parkingSpotId", label: "location", sortable: true, class: "text-center" },
        { key: "AmountOwed", label: "balance", sortable: true, class: "text-center" },

        {
          sortable: true,
          sortByFormatted: true,
          filterByFormatted: true,
        },
        { key: "actions", label: "Actions", class: "text-center" },
      ],
      perPage: 1000, // this is the max number of cars displayed in the list
      sortBy: "",
      sortDesc: false,
      sortDirection: "asc",
      filter: null,
      filterOn: [],
      parkingSpotForm: false,
      ticketId: "",
      request: "",
      hideAllButtons: false,
    };
  },
  computed: {
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter((f) => f.sortable)
        .map((f) => {
          return { text: f.label, value: f.key };
        });
    },
  },
  methods: {
    info(item, index, button) {
      this.infoModal.title = `Row index: ${index}`;
      this.infoModal.content = JSON.stringify(item, null, 2);
      this.$root.$emit("bv::show::modal", this.infoModal.id, button);
    },
    calcTime(date) {
      var now = moment(new Date());
      var end = moment(date);
      var duration = moment.duration(now.diff(end));
      var minutes = duration.asMinutes();
      var calc = (minutes / 60) * 5;
      return calc.toFixed(2);
    },

    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
    showParkingSpotForm(ticketId, showLabels) {
      this.ticketId = ticketId;
      this.request = showLabels;
      this.parkingSpotForm = true;
      this.hideAllButtons = !showLabels ? true : false;
    },
    UpdateCars(){
      ValetService.getAllTheInfo().then((response) => {
      this.$store.commit("LOAD_CAR_LIST", response.data);
      
      //alert("what up?")
    });
    this.$forceUpdate();
    },
    cancelButton() {
      this.hideAllButtons=false;
      this.showParkingSpotForm=false;
    },
    tryOut(){
      
      alert("hey there!");
    }
  },
};
</script>

<style>
.attributesList {
  color: rgb(80, 80, 80);
}

#displayPatronList {
  background-color: rgb(238, 238, 238);
  padding: 1%;
  border-radius: 0.5rem;
}

#displayCarList {
  background-color: rgb(250, 250, 250);
  padding: 1%;
  border-radius: 0.5rem;
  margin-top: 1%;
}

#listStuffModal {
  list-style: none;
}

#filterCarDeets {
  font-size: 4vh !important;
}

#dodge {
  font-family: "Jua", "Times New Roman", Times, serif !important;
  font-size: 2.5vh;
}

tr:only-child {
  text-transform: uppercase;
}
</style>