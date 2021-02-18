<template>
  <div>
    <h3
      v-if="
        this.valetSelection == 'pickupCar' && ShowLabels ||
        this.patronSelection == 'pickupCar'
      "
    >
      REQUEST CAR PICKUP
    </h3>
    <h3 v-if="ShowLabels && valetSelection == 'checkoutCar'" >CHECK-OUT</h3>
    <h3 v-if="this.patronSelection == 'showBalance'">VIEW BALANCE</h3>
    
      <b-form @submit="PatronCarDetailsSubmit" @reset="onReset" v-if="show" class="vertical-buttons">
        <b-col sm="3">
          
        <b-form-group
          id="input-group-1"
          label="Enter Valet Slip Number:"
          label-for="input-1"
          description="Please enter your valet slip number."
          v-if="ShowLabels"
        >
        
          <b-form-input
            id="input-1"
            v-model="form.valetSlipNumber"
            type="text"
            required
            placeholder="Enter Valet Slip Number"
            
          >
          </b-form-input>
        </b-form-group>
        <b-form-input
            id="input-1"
            v-model="form.valetSlipNumber"
            type="text"
            required
            placeholder="Enter Valet Slip Number"
            v-if="!ShowLabels"
          >
          </b-form-input>
      </b-col>
      <b-row align-v="center" >
        <b-col sm="3">
          <b-button type="submit" variant="primary">Submit</b-button>
        </b-col>
        <b-col sm="3">
          <b-button type="reset" variant="warning">Reset</b-button>
        </b-col>
        <b-col sm="3">
            <b-button @click="onCancel, $emit('click-cancel')" variant="danger" class="h-75">Cancel</b-button>
        </b-col>
      </b-row>
    </b-form>

    <h3 v-if="showValetCall">The valet will arrive shortly with your car.</h3>
    <div>
      <amount-owed v-if="showAmountOwed"
      v-bind:amountOwed="this.finalAmountOwed"
      v-on:complete-checkout="$emit('complete-checkout')"
      />
    </div>
     <div v-if="showValetRequestMessage" class="vertical-buttons">       
        <b-col>
          <h3>Pickup request from Valet received</h3>
        </b-col>
        <b-col sm="3">
            <b-button @click="$emit('update-requested-cars')" variant="success" class="h-75">Ok</b-button>
        </b-col>
     </div>
    <patron-car-details v-if="showPatronCar" v-bind:slipId="slipId" />
  </div>
</template>   

<script>
import PatronCarDetails from "@/components/PatronCarDetails.vue";
import PatronService from "@/services/PatronService.js";
import CarDetailsService from "@/services/CarDetailsService.js";
import ParkingService from "@/services/ParkingLotService.js";
import AmountOwed from './AmountOwed.vue';

export default {
  slipId: "",
  
  props: ["patronSelection", "valetSelection", "request", "ticketId", "fromCarList"],
  data() {
    return {
      form: {
        valetNumber: "",
        name: "",
        checked: [],
      },
      show: true,
      showValetCall: false,
      showPatronCar: false,
      finalAmountOwed: "",
      showAmountOwed: false,
      showValetRequestMessage: false,
      
      
    };
  },
  components: { PatronCarDetails, AmountOwed },
  watch: {
    fromCarList: {
      immediate: true,
      handler: 'SkipSlipIDEntry'
    }    
  },  
  computed: {
    ShowLabels() {
      
      if (this.valetSelection == 'checkoutCar' || this.valetSelection == 'pickupCar') {
        //checkoutCar can come from ListOfCars or ValetView
        if (this.fromCarList) {
          //request from ListOfCars will have ticketId
          //so go straight to amount Owed screen          
          return false
        }
        if (this.request == undefined || this.request) {          
          return true
        } else {
          return false
        }
      } else {
        return false
      }                       
    },
    ShowAmountOwedScreen() {
      //console.log(this.showAmountOwed)
      if (this.showAmountOwed || this.fromCarList) {
        if (this.valetSelection == 'pickupCar') {
          return false;
        } else {
          return true
        }
      } else {
        return false;
      }
      
    }
  },
  methods: {
    PatronCarDetailsSubmit(evt) {
      if (!this.fromCarList) {
        evt.preventDefault();
      }      
      PatronService.getValetSlip(this.form.valetSlipNumber).then((response) => {
        if (response.data != "") {
          //as long as ID exists, do this . .
          if (this.patronSelection == "showBalance") {
            this.showPatronCar = !this.showPatronCar;
            this.show = false;
            this.slipId = this.form.valetSlipNumber;
          } else if (this.patronSelection == "pickupCar") {
            this.showValetCall = !this.showValetCall;
            this.show = false;
            CarDetailsService.changeParkingSpotStatus(
              this.form.valetSlipNumber
            ).then((response) => {
              //console.log(response.status);
            });
            //verify that the Valet Slip ID exists***
          } else if (this.valetSelection == "pickupCar") {
            CarDetailsService.changeParkingSpotStatus(
              this.form.valetSlipNumber
            );
            this.showValetRequestMessage = !this.showValetRequestMessage;
            this.show = false;
          } else if (this.valetSelection == "checkoutCar") {            
            CarDetailsService.checkoutCar(this.form.valetSlipNumber).then(
              (response) => {
                this.finalAmountOwed =
                  "$" + response.data.amountOwed.toFixed(2);
                (this.show = false), (this.showAmountOwed = true);                
              }
              
            );
            
          }
        } else {
          //if id doesn't exist/entered incorrectly do this
          alert("Please enter a valid slip number");
        }
      });
    },
    SkipSlipIDEntry() {
      if (this.fromCarList) {
        
        this.form.valetSlipNumber=this.ticketId;
        this.PatronCarDetailsSubmit(this.form.name);   
        
      }
    },
    onSubmitPatronRequest() {
      this.show = false;
    },
    UpdateParkingLot(){
      //this function call very similar to created method in Parking Lot component
      ParkingService.getParkingSpots()
      .then((response) => {
        
        this.$store.commit("FILL_PARKING_SPOTS", response.data);
        
        //this.loadingLotData = true;
      })      
    },
    onReset(evt) {
      evt.preventDefault();
      // Reset our form values
      this.form.valetSlipNumber = "";
      // Trick to reset/clear native browser form validation state
      this.show = false;
      this.$nextTick(() => {
        this.show = true;
      });
    },
    onCancel(evt) {
      evt.preventDefault();
      this.form.slipId = "";
      this.show = false;
    }
  },
};
</script>

<style>
</style>

