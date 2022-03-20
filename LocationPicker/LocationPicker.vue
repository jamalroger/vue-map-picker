<template>
  <div class="LocationPicker">
    <div class="LocationPicker__map" ref="map" style="width: 100%"></div>
    <input
      type="text"
      class="LocationPicker__autocomplete"
      ref="input"
      placeholder="Search by adresse"
    />
    <info-window class="LocationPicker__info-window" ref="info"></info-window>
    <v-icon
      @click="getGeoLocation()"
      color="primary"
      style="z-index: 99; bottom: 80px; right: 5px; position: absolute"
      size="40"
      >fa-street-view</v-icon
    >
  </div>
</template>


<script>
import InfoWindow from "./InfoWindow.vue";
import { Capacitor } from "@capacitor/core";
import { Geolocation } from "@capacitor/geolocation";
export default {
  props: {
    value: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      geocoder: null,
      map: null,
      marker: null,
      infoWindow: null,
      autocomplete: null,
    };
  },
  emits: ["update:position"],
  components: { InfoWindow },
  computed: {
    google() {
      return window.google;
    },
  },
  mounted() {
    this.locationPickerInit({});
  },
  methods: {
    locationPickerInit() {
      this.geocoder = new this.google.maps.Geocoder();
      this.map = new this.google.maps.Map(this.$refs.map, {
        center: { lat: 0, lng: 0 },
        zoom: 5,
        disableDefaultUI: true,
      });

      this.marker = new this.google.maps.Marker({
        map: this.map,
        position: this.map.getCenter(),
        draggable: true,
      });

      this.infoWindow = new this.google.maps.InfoWindow({
        content: this.$refs.info.$el,
      });

      this.autocomplete = new this.google.maps.places.Autocomplete(
        this.$refs.input,
        {
          types: ["geocode"],
        }
      );

      this.map.controls[this.google.maps.ControlPosition.TOP_LEFT].push(
        this.$refs.input
      );

      // events
      this.google.maps.event.addListenerOnce(
        this.map,
        "idle",
        this.openInfoWindow
      );

      this.marker.addListener("dragstart", this.closeInfoWindow);
      this.marker.addListener("dragend", this.geocodeLocation);
      this.autocomplete.addListener("place_changed", this.moveMarker);
    },

    openInfoWindow() {
      this.infoWindow.open(this.map, this.marker);
    },
    closeInfoWindow() {
      this.infoWindow.close();
    },
    geocodeLocation(e) {
      this.map.panTo(e.latLng);
      this.$refs.input.value = "";
      this.setPosition({ lng: e.latLng.lng(), lat: e.latLng.lat() });
    },
    getLocation(latLng) {
      return new Promise((reslove, reject) => {
        this.geocoder.geocode({ latLng: latLng }, (response) => {
          if (response && response.length > 0) {
            reslove(response[0]);
          } else {
            reject(0);
          }
        });
      });
    },
    moveMarker() {
      var place = this.autocomplete.getPlace();
      var location = place.geometry && place.geometry.location;
      if (location) {
        this.setPosition({ lng: location.lng(), lat: location.lat() }, 17);
      }
    },
    async getGeoLocation() {
      if (Capacitor.isNativePlatform()) {
        const position = await Geolocation.getCurrentPosition();
        this.setPosition(
          {
            lng: position.coords.longitude,
            lat: position.coords.latitude,
          },
          17
        );
      } else {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition((position) => {
            this.setPosition(
              {
                lng: position.coords.longitude,
                lat: position.coords.latitude,
              },
              17
            );
          });
        }
      }
    },
    async setPosition(position, zoom = null) {
      let adresse = await this.getLocation(position);
      let palce = adresse.formatted_address
        .split(",")
        .map((item) => item.trim())
        .join(",");

      this.$emit("input", {
        adresse: palce,
        lat: position.lat,
        lng: position.lng,
      });

      this.map.panTo(position);
      this.marker.setPosition(position);
      if (zoom) this.map.setZoom(zoom);

      this.$refs.info.showAddress(adresse);
      this.openInfoWindow();
    },
  },
};
</script>


<style>
.LocationPicker,
.LocationPicker__map {
  height: 100%;
}
.LocationPicker__autocomplete {
  padding: 12px 14px;
  margin: 2%;
  min-width: 300px;
  font-size: 15px;
  font-weight: 300;
  text-overflow: ellipsis;
  border: 0;
  border-radius: 13px;
  background: #fff;
  width: -webkit-fill-available;
  font-family: "helvetica";
  color: var(--v-secondary-base);
  border: 1px solid var(--v-secondary-base);
  padding-left: 36px;
}
.LocationPicker > .LocationPicker__autocomplete,
.LocationPicker > .LocationPicker__info-window {
  display: none;
}
</style>