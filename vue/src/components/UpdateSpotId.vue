<template>
  <div>
    <b-form @submit.prevent="UpdateSpotNumber" @reset="onCancel" v-if="show" :class="this.request ? 'horizontal-buttons' : 'vertical-buttons'">
      <b-form-group
        id="Spot Label"
        label="Enter Spot Number:"
        label-for="Spot Label"
        description="Enter Spot Number"
        v-if="this.request"
      >
      </b-form-group>
      <b-col sm="3">
        <b-form-input
          id="Spot Number Input"
          v-model="form.spotNumber"
          type="text"
          required
          placeholder="Spot Number"          
        ></b-form-input></b-col>
      
      
      <b-col :sm="this.request ? '3' : '2'">
      <b-button type="submit" variant="primary" @click="$emit('spot-update')">Submit</b-button>
      <!--spot-update is in List Of Cars component-->
      </b-col>
      <b-col :sm="this.request ? '3' : '2'">
      <b-button type="reset" variant="danger" @click="$emit('unhide-buttons')">Cancel</b-button>
      <!--unhide-buttons is in List Of Cars component--></b-col>
      
    </b-form>

    <div v-if="showSpotUpdatedMessage" class="vertical-buttons">
      <b-col>
      <h3 >Spot Number Updated</h3>
      </b-col>
      <b-col sm="3">
      <b-button variant="primary" @click="$emit('unhide-buttons')">Ok</b-button>
      </b-col>
    </div>

  </div>

</template>

<script>
import ValetService from "@/services/ValetService.js";
import ParkingService from "@/services/ParkingLotService.js";

export default {
  name: "update-spot-id",
  
  props: ["ticketId", "request"],
  //ticketId bound from ListOfCars comp
  data() {
    return {
      form: {
        spotNumber: "",
        name: "",
      },
      show: true,
      showValetCall: false,
      showPatronCar: false,
      showValetRequestMessage: false,
      showSpotUpdatedMessage: false,
      
    };
  },
  /*created() {
      this.show=this.parkingSpotForm;
  },
  watch: {
    show: function() {
      return true;
      //return this.parkingSportForm;
    }
  },*/
  
  methods: {
    UpdateSpotNumber() {
      if (this.form.spotNumber < 1 || this.form.spotNumber > 10) {
        alert("Please enter a valid spot number");
      } else {
        this.$store.state.parkingSpots.forEach((spot) => {
          if (spot.parkingSpotId == this.form.spotNumber) {
            if (spot.isOccupied == true) {
              alert("Parking spot already taken");
            } else {
              ValetService.updateParkingSpot(
                this.ticketId,
                this.form.spotNumber
              ).then((response) => {
                if (response.status == 201 || response.status == 200) {
                  this.show = false;
                  this.showSpotUpdatedMessage = true;
                  
                  console.log(this.parkingLotSpots);
                  //this.UpdateParkingLot();
                  this.UpdateParkingLot();
                  
                  this.UpdateCar();//contains updated spot car is in
                  
                  //alert(response.status);
                  //location.reload();
                } else {
                  alert(response.status + " " + this.form.spotNumber + " for " + spot.parkingSpotId + " on " +this.ticketId + " because isOccupied equals " + spot.isOccupied);
                }
              });
              
            }
          }
        });
      }
    },
    UpdateParkingLot(){
      //this function call very similar to created method in Parking Lot component
      ParkingService.getParkingSpots()
      .then((response) => {
        
        this.$store.commit("FILL_PARKING_SPOTS", response.data);
        //this.loadingLotData = true;
      })      
    },
    UpdateCar() {
      //same call made in List Of Cars component (UpdateCars())
      ValetService.getAllTheInfo().then((response) => {
      this.$store.commit("LOAD_CAR_LIST", response.data); 
      })     
    },
    
    onCancel(evt) {
      evt.preventDefault();
      // Reset our form values
      this.form.spotNumber="";
      
      // Trick to reset/clear native browser form validation state
      this.show = false;
      /*this.$nextTick(() => {
        this.show = true;
      });*/
    },
  },
};
</script>
<style>
.vertical-buttons{
  display: flex;
}
.horizontal-buttons{
  display: flex;
  flex-direction: column;
  
}
</style>