<template>
  <div class="container-fluid mt-5">
    <div class="row">
      <div class="col col-md-5 px-0 map-col mr-4">
        <h2 className="text-right">START YOUR AWESOME PLAN</h2>
        <gmap-map
          ref="mapRef"
          :options="{
            zoomControl: true,
            mapTypeControl: false,
            scaleControl: false,
            streetViewControl: true,
            rotateControl: false,
            fullscreenControl: true,
            disableDefaultUi: false,
            zoomControlOptions: { position: 1 },
            streetViewControlOptions: { position: 5 }
          }"
          :center="center"
          :zoom="10"
          map-type-id="roadmap"
          style="width:100%;height: 55vh;"
          @center_changed="updateCenter"
          @idle="updateData"
        >
        </gmap-map>

        <div class="row mt-4">
          <div class="col">
            <gmap-autocomplete
              class="form-control"
              placeholder="Search destination"
              :value="searchMap"
              :options="autocompleteOptions"
              @place_changed="setPlace"
            >
            </gmap-autocomplete>
          </div>
          <div class="col">
            <button class="btn btn-danger mr-2" @click="resetMap">
              Reset Trip
            </button>
            <button class="btn btn-dark" @click="optimizeRoute">
              Optimize Trip
            </button>
          </div>
        </div>
      </div>
      <!--/col-md-4-->
      <div class="col card-col">
        <div class="row">
          <div class="card">
            <div class="card-body">
              <div class="row">
                <div class="col col-md-5 offset-md-2">
                  <h4 class="mb-4">Tempat wisata terpopuler</h4>
                  <draggable
                    class="card-col-attraction"
                    :sort="false"
                    v-model="places"
                    v-bind="dragOptions"
                  >
                    <transition-group
                      mode="out-in"
                      enter-active-class="animate__animated animate__backInDown"
                      leave-active-class="animate__animated animate__backOutDown animate__faster"
                    >
                      <div
                        v-for="value in places"
                        :key="value.name"
                        class="col col-12 px-2 mb-3"
                      >
                        <left-card :value="value"></left-card>
                      </div>
                      <!--/col-12-->
                    </transition-group>
                  </draggable>
                  <!--/card-col-attraction-->
                </div>
                <div class="col col-5">
                  <h4 class="mb-4">Pilih tempat wisata</h4>
                  <draggable
                    class="card-col-attraction"
                    v-model="places_selected"
                    v-bind="dragOptions"
                    @change="changePlaceSelect"
                  >
                    <div
                      v-for="value in places_selected"
                      :key="value.name"
                      class="col col-12 px-2 mb-3"
                    >
                      <right-card :value="value"></right-card>
                    </div>
                    <!--/col-12-->
                  </draggable>
                  <!--/card-col-attraction-->
                </div>
                <!--/col-->
              </div>
              <!--/row-->
            </div>
            <!--/card-body-->
          </div>
          <!--/card-->
        </div>
        <!--/row-->
      </div>
      <!--/col card-col-->
    </div>
    <!--/row-->
  </div>
</template>

<script>
import { gmapApi } from "vue2-google-maps";
import draggable from "vuedraggable";

import LeftCard from "./CardComponent/LeftCard.vue";
import RightCard from "./CardComponent/RightCard.vue";

export default {
  data() {
    return {
      searchMap: null,
      autocompleteOptions: {
        componentRestrictions: {
          country: ["id"]
        }
      },
      center: { lat: -8.381357822670871, lng: 115.13967209436002 },
      current_position: { lat: null, lng: null },
      distanceTo: { atm: null, pharmacy: null, convenience_store: null },
      places: [],
      places_selected: []
    };
  },
  computed: {
    google: gmapApi,
    mapObject() {
      return this.$refs["mapRef"].$mapObject;
    },
    directionsRenderer() {
      return new this.google.maps.DirectionsRenderer();
    },
    directionsService() {
      return new this.google.maps.DirectionsService();
    },
    dragOptions() {
      return {
        animation: 200,
        group: "places",
        disabled: false,
        ghostClass: "ghost"
      };
    }
  },
  methods: {
    callbackAtm(results, status) {
      if (status == this.google.maps.places.PlacesServiceStatus.OK) {
        this.distanceTo.atm = results[0].geometry.location;
      }
    },
    callbackPharmacy(results, status) {
      if (status == this.google.maps.places.PlacesServiceStatus.OK) {
        this.distanceTo.pharmacy = results[0].geometry.location;
      }
    },
    callbackConvenienceStore(results, status) {
      if (status == this.google.maps.places.PlacesServiceStatus.OK) {
        this.distanceTo.convenience_store = results[0].geometry.location;
      }
    },
    changePlaceSelect(e) {
      this.calculateAndDisplayRoute();

      if (e.added) {
        let map = new this.google.maps.places.PlacesService(this.mapObject);
        for (let key in this.places_selected) {
          //location for search nearby
          let location = this.places_selected[key].position;
          // search nearby atm from location
          map.nearbySearch(
            {
              location: location,
              rankBy: this.google.maps.places.RankBy.DISTANCE,
              type: ["atm"]
            },
            this.callbackAtm
          );
          // search nearby pharmacy from location
          map.nearbySearch(
            {
              location: location,
              rankBy: this.google.maps.places.RankBy.DISTANCE,
              type: ["pharmacy"]
            },
            this.callbackPharmacy
          );
          // search nearby convenience_store from location
          map.nearbySearch(
            {
              location: location,
              rankBy: this.google.maps.places.RankBy.DISTANCE,
              type: ["convenience_store"]
            },
            this.callbackConvenienceStore
          );

          // wait 1 second until distanceTo ready
          setTimeout(() => {
            this.places_selected[key].atm = (
              this.getDistance(location, this.distanceTo.atm) / 1000
            ).toFixed(2);
            this.places_selected[key].pharmacy = (
              this.getDistance(location, this.distanceTo.pharmacy) / 1000
            ).toFixed(2);
            this.places_selected[key].convenience_store = (
              this.getDistance(location, this.distanceTo.convenience_store) /
              1000
            ).toFixed(2);
          }, 1000);
        }
      }
    },
    setPlace(e) {
      this.mapObject.setCenter(e.geometry.location);
    },
    rad(x) {
      return (x * Math.PI) / 180;
    },
    getDistance(p1, p2) {
      /* Haversine formula */
      let R = 6378137; // Earth’s mean radius in meter
      let dLat = this.rad(p2.lat() - p1.lat());
      let dLong = this.rad(p2.lng() - p1.lng());
      let a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(this.rad(p1.lat())) *
          Math.cos(this.rad(p2.lat())) *
          Math.sin(dLong / 2) *
          Math.sin(dLong / 2);
      let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      let d = R * c;
      return d; // returns the distance in meter
    },
    optimizeRoute() {
      let data = this.places_selected;

      if (data.length > 2) {
        for (let i = 0; i < data.length; i++) {
          for (let j = 1; j < data.length; j++) {
            try {
              let distance_one = (
                this.getDistance(data[i].position, data[j].position) / 1000
              ).toFixed(2);
              let distance_two = (
                this.getDistance(data[i].position, data[j + 1].position) / 1000
              ).toFixed(2);
              let tmpData = null;
              if (distance_one > distance_two) {
                tmpData = data[j];
                data[j] = data[j + 1];
                data[j + 1] = tmpData;
              }
            } catch (err) {
              this.calculateAndDisplayRoute();
              return;
            }
          }
        }
      } else alert("minimal 3 data");
    },
    resetMap() {
      this.directionsRenderer.setDirections({ routes: [] });
      this.places_selected = [];
      setTimeout(() => {
        this.mapObject.setCenter(this.center);
        this.mapObject.setZoom(10);
      }, 500);
    },
    calculateAndDisplayRoute() {
      this.directionsRenderer.setMap(this.mapObject);

      let data = this.places_selected;

      if (data.length > 2) {
        let start = data[0].position;
        let end = data[data.length - 1].position;

        let raw_waypts = data.slice(1, data.length - 1);
        let waypts = [];

        for (let key in raw_waypts) {
          waypts.push({
            location: raw_waypts[key].position,
            stopover: true
          });
        }
        // set direction to map
        let direction = this.directionsRenderer;
        this.directionsService.route(
          {
            origin: start,
            destination: end,
            waypoints: waypts,
            optimizeWaypoints: true,
            travelMode: this.google.maps.TravelMode.DRIVING
          },
          function(response, status) {
            if (status === this.google.maps.DirectionsStatus.OK) {
              direction.setDirections(response);
            }
          }
        );
      }
    },
    updateCenter(e) {
      this.current_position.lat = e.lat();
      this.current_position.lng = e.lng();
    },
    callbackTouristAttraction(results, status) {
      if (status == this.google.maps.places.PlacesServiceStatus.OK) {
        //reset data when map idle and status ok
        this.places = [];

        for (let key in results) {
          let value = results[key];

          if (value.user_ratings_total >= 50) {
            let tmp = {
              name: value.name,
              position: value.geometry.location,
              rating: value.rating,
              user_rating: value.user_ratings_total,
              address: value.plus_code.compound_code
                .split(" ")
                .slice(1)
                .join(" "),
              icon: value.photos[0].getUrl()
            };
            // check duplicate places in places selected
            let found = this.places_selected.some(el => el.name === tmp.name);
            if (!found) this.places.push(tmp);
          }
        }
      }
    },
    updateData() {
      let current_cursor = new this.google.maps.LatLng(
        this.current_position.lat,
        this.current_position.lng
      );
      // object map
      let map = new this.google.maps.places.PlacesService(this.mapObject);
      // search nearby tourist attraction from current cursor
      map.nearbySearch(
        {
          location: current_cursor, //Add initial lat/lon here
          rankBy: this.google.maps.places.RankBy.DISTANCE,
          type: ["tourist_attraction"]
        },
        this.callbackTouristAttraction
      );
    }
  },
  components: {
    draggable,
    RightCard,
    LeftCard
  },
  mounted() {
    setTimeout(() => {
      this.current_position.lat = this.center.lat;
      this.current_position.lng = this.center.lng;
      this.updateData();
    }, 1000);
  }
};
</script>

<style>
@import "../assets/fontawesome/css/all.css";

.gm-ui-hover-effect {
  display: none !important;
}
.gm-style-iw-d {
  overflow: hidden !important;
}
.gm-style .gm-style-iw-c {
  padding: 0px;
  top: 40px;
  border-radius: 12px;
}
.gm-style .gm-style-iw-t::after {
  top: 38px;
}
.gmnoprint > div {
  border-radius: 8px !important;
}
.gm-control-active.gm-fullscreen-control {
  border-radius: 8px !important;
}
.gm-style-pbc {
  opacity: 0 !important;
}
.img-fit {
  object-fit: cover;
}
.fs-10 {
  font-size: 10px;
}
.fs-11 {
  font-size: 11px;
}
.fs-12 {
  font-size: 12px;
}
.card-name {
  font-size: 20px;
  line-height: 30px;
  max-height: 60px;
  color: #000;
  margin-bottom: 0;
  justify-content: space-between;
  letter-spacing: 0.25px;
  line-height: 30px;
}
.card-address {
  color: #757575;
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
  max-height: 48px;
}
.map-col {
  left: 5vh;
  z-index: 1;
}
.card-col {
  left: -5vh;
}
.card-col-attraction {
  max-height: 500px;
  overflow: auto;
}
.card-rating {
  color: #757575;
  font-size: 12px;
  font-weight: 400;
  line-height: 18px;
  max-height: 48px;
  display: flex;
}
.vue-star-rating {
  height: 12px;
}
.ghost {
  opacity: 0.5;
}
.text-grey {
  color: #757575;
}
</style>
