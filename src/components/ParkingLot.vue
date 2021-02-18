<template>
  <div v-if="!loadingLotData">
    <div id="home-parking-lot-container-yes">

      <div class="container-text">
            <ul>
              <li>
                <span class="hr2">Available&nbsp;
                <b-icon class="hr4" icon="circle-fill" />
                </span>
              </li>
              <li>
                <span class="hr2">&nbsp;&nbsp;Occupied&nbsp;
                <b-icon class="hr5" icon="circle-fill" />
                </span>
              </li>
            </ul>
      </div>
      <div>
                  
        <h3 class="container-text">Available Spots</h3>
        <b-alert
          id="alert-full"
          style="text-align: center"
          show
          variant="danger"
          v-if="isFull"
          >Parking Lot Full!</b-alert
        >
      </div>
        <div v-if="!isFull">
          <div class="parking-lot-container">
            <parking-spot
              v-for="parkingSpot in this.$store.state.parkingSpots"
              :key="parkingSpot.parkingSpotId"
              v-bind:parking-spot="parkingSpot"
            />
          </div>
          
        </div>
      </div>
    </div>
  
</template>

<script>
import ParkingService from "@/services/ParkingLotService.js";
import ParkingSpot from "@/components/ParkingSpot.vue";

export default {
  data() {
    return {
      parkingSpots: [],
      loadingLotData: true,
    };
  },
  components: { ParkingSpot },
  name: "parking-lot",

  created() {
    ParkingService.getParkingSpots()
      .then((response) => {
        // this.parkingSpots = response.data;
        this.$store.commit("FILL_PARKING_SPOTS", response.data);
        this.loadingLotData = true;
      })
      .finally(() => {
        this.loadingLotData = false;
      });
  },
  computed: {
    isFull() {
      let occupiedSpots = 0;
      this.$store.state.parkingSpots.forEach((spot) => {
        if (spot.isOccupied) {
          occupiedSpots++;
        }
      });

      return this.$store.state.parkingSpots.length == occupiedSpots;
    },
  },
};
</script>

<style>
#home-parking-lot-container-yes {
  color: white;
  background-image: url("../img/p-lot-background.jpg");
  padding-top: 0;
  padding-bottom: 0.05%;
  font-family: "Jua";
  text-transform: uppercase;
  text-align: center;
}
#alert-full {
  font-size: 2.5rem !important;
  font-weight: 800 !important;
  letter-spacing: 1px !important;
  margin-bottom: 4%;
}
#p-lot-container-header {
  color: white;
  background-image: url("../img/p-lot-background.jpg");
}
#p-lot-parent-container {
  grid-template-rows: 1fr 2fr;
}
.container-text ul{
  display: flex;
  
  min-width: 10%;
  justify-content: flex-start;
}
.container-text li {
  list-style: none;
  text-align: left;
  font-size: 90%;
  color: rgb(230, 230, 230);
  padding-right: 20px;
}
.parking-lot-container {
  color: white;
  display: flex;
  justify-content: space-evenly;
  flex-wrap: wrap;
  align-items: center;
}
</style>

