<template>
  <v-flex md9>
    <l-map
      v-if="jsonResult.routesList"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height:90%"
    >
      <l-tile-layer :url="url" :attribution="attribution" />

      <!-- origin -->
      <l-marker
        :lat-lng="
            latLng(
              jsonResult.routesList[0].origin.latitude,
              jsonResult.routesList[0].origin.longitude
            )
          "
      >
        <l-icon
          :lat-lng="
              latLng(
                jsonResult.routesList[0].origin.latitude,
                jsonResult.routesList[0].origin.longitude
              )
            "
          :icon-size="iconSize"
          :icon-url="iconUrl"
        />

        <l-tooltip :options="{ permanent: false, interactive: true }">
          <div @click="innerClick">
            <h4 style="text-decoration:underline">origin - Central de distribucion</h4>
            <br />
            <h4>description:</h4>
            {{ jsonResult.routesList[0].origin.description }}
            <br />
            <h4>address:</h4>
            {{ jsonResult.routesList[0].origin.address }}
            <h4>Coordenadas:</h4>
            {{ latLng(jsonResult.routesList[0].origin.latitude,
            jsonResult.routesList[0].origin.longitude) }}
          </div>
        </l-tooltip>
      </l-marker>
      <!-- END origin -->

      <!-- Paradas -->
      <div :key="i" v-for="(route, i) in jsonResult.routesList">
        <div :key="j" v-for="(parada, j) in route.stopsList">
          <l-marker :lat-lng="latLng(parada.latitude, parada.longitude)">
            <l-tooltip :options="{ permanent: false, interactive: true }">
              <div @click="innerClick">
                <h4 style="text-decoration:underline">Parada</h4>
                <br />
                <h4>route:</h4>
                {{ route.id }}
                <br />
                <h4>Hora de llegada:</h4>
                {{ getArrivalTime(parada.startDate) }} -
                {{ getRelativeTime(parada.startDate) }}
                <h4>Coordenadas:</h4>
                {{ latLng(parada.latitude, parada.longitude) }}
                <h4>sequence:</h4>
                {{ parada.sequence }}
              </div>
            </l-tooltip>
          </l-marker>

          <!-- deliveries -->
          <div :key="k" v-for="(entrega, k) in parada.deliveriesList">
            <l-circle-marker
              v-if="seeDeliveries"
              :lat-lng="latLng(entrega.latitude, entrega.longitude)"
              :radius="circle.radius"
              :fillColor="circle.color"
              :fillOpacity="1"
              :color="circle.color"
            >
              <l-tooltip v-if="seeDeliveries" :options="{ permanent: false, interactive: true }">
                <div @click="innerClick">
                  <h4 style="text-decoration:underline">Entrega</h4>
                  <br />
                  <h4>Cliente:</h4>
                  {{ entrega.description }}
                  <h4>address:</h4>
                  {{ entrega.address }}
                  <h4>Hora de llegada:</h4>
                  {{ getArrivalTime(entrega.startDate) }} -
                  {{ getRelativeTime(entrega.startDate) }}
                  <h4>Coordenadas:</h4>
                  {{ latLng(entrega.latitude, entrega.longitude) }}
                  <h4>sequence:</h4>
                  {{ entrega.sequence }}
                </div>
              </l-tooltip>
            </l-circle-marker>

            <l-polyline
              v-if="seeDeliveries"
              :lat-lngs="getDeliveriesCoordinates(parada.deliveriesList)"
              :color="'black'"
            ></l-polyline>
          </div>

          <!-- END ENTREGA -->

          <l-polyline
            :lat-lngs="getStopsCoordinates(route.stopsList)"
            :color="'#' + ((Math.random() * 0xffffff) << 0).toString(16)"
          ></l-polyline>
        </div>
        <!-- End Parada -->
      </div>
    </l-map>
  </v-flex>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { latLng, circleMarker, Icon } from "leaflet";
import {
  LMap,
  LTileLayer,
  LMarker,
  LTooltip,
  LPolyline,
  LIcon,
  LCircleMarker
} from "vue2-leaflet";
import paradaIcon from "../assets/2x/sharp_local_shipping_black_18dp.png";

var moment = require("moment");

// this part resolve an issue where the markers would not appear
delete Icon.Default.prototype._getIconUrl;

Icon.Default.mergeOptions({
  iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
  iconUrl: require("leaflet/dist/images/marker-icon.png"),
  shadowUrl: require("leaflet/dist/images/marker-shadow.png")
});

export default {
  name: "Map",

  props: {
    jsonResult: null,
    seeDeliveries: false
  },

  components: {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LPolyline,
    LIcon,
    LCircleMarker
  },

  data: function() {
    return {
      zoom: 13,
      center: latLng(
        this.jsonResult.routesList[0].origin.latitude,
        this.jsonResult.routesList[0].origin.longitude
      ),
      url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(
        this.jsonResult.routesList[0].origin.latitude,
        this.jsonResult.routesList[0].origin.longitude
      ),
      withTooltip: latLng(
        this.jsonResult.routesList[0].origin.latitude,
        this.jsonResult.routesList[0].origin.longitude
      ),
      currentZoom: 11.5,
      currentCenter: latLng(
        this.jsonResult.routesList[0].origin.latitude,
        this.jsonResult.routesList[0].origin.longitude
      ),
      showParagraph: false,
      mapOptions: {
        attributionControl: false,
        zoomSnap: true
      },
      circle: {
        radius: 4,
        color: "red"
      },
      colorCache: {},
      imperial: false,
      showMap: true,
      iconUrl: paradaIcon,
      iconSize: [36, 41]
    };
  },

  methods: {
    latLng: function(lat, lng) {
      return latLng(lat, lng);
    },
    getStopsCoordinates(stopsList) {
      return [
        [
          this.jsonResult.routesList[0].origin.latitude,
          this.jsonResult.routesList[0].origin.longitude
        ],
        ...stopsList.map(x => [x.latitude, x.longitude])
      ];
    },
    getDeliveriesCoordinates(deliveriesList) {
      return [...deliveriesList.map(x => [x.latitude, x.longitude])];
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    },
    dynamicSize() {
      return [this.iconSize, this.iconSize * 1.15];
    },
    dynamicAnchor() {
      return [this.iconSize / 2, this.iconSize * 1.15];
    },
    getArrivalTime(date) {
      return (
        moment(date)
          .hours()
          .toString() +
        ":" +
        moment(date)
          .minutes()
          .toString()
      );
    },
    getRelativeTime(date) {
      return moment(date).fromNow();
    }
  }
};
</script>
