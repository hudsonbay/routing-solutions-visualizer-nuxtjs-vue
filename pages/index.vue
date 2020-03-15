<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-select
        v-if="jsonExists"
        v-bind:items="getDates"
        v-model="selectedDate"
        label="Filter by date"
        single-line
        class="shrink mr-3"
        bottom
        @input="filterRoutesBasedOnDateSelection"
      ></v-select>

      <v-btn info v-if="jsonExists" v-on:click="invertDeliveriesMode">
        {{
        button.text
        }}
      </v-btn>

      <v-spacer></v-spacer>

      <v-btn info color="success" v-if="jsonExists" v-on:click="showRoutesAgain">All routes</v-btn>

      <v-spacer></v-spacer>

      <input type="file" @change="onFileSelected" />
    </v-app-bar>

    <v-content :height="windowHeight">
      <v-container fluid grid-list-md>
        <div>
          <v-layout>
            <v-flex>
              <v-layout row>
                <Kpi
                  :jsonResult="jsonToShow"
                  @isolate-route="isolateRoute"
                  v-if="jsonExists"
                  :seeDeliveries="seeDeliveries"
                />
                <Map
                  style="height:550px"
                  v-if="jsonExists"
                  :jsonResult="jsonOnMap"
                  :seeDeliveries="seeDeliveries"
                />
              </v-layout>
            </v-flex>
          </v-layout>
        </div>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import Map from "~/components/Map.vue";
import Kpi from "~/components/Kpi.vue";
var moment = require("moment");

export default {
  components: {
    Map,
    Kpi
  },

  data: () => ({
    jsonResult: [],
    jsonToShow: [],
    selectedDate: "",
    seeDeliveries: false,
    windowHeight: "",
    button: {
      text: "See stops + deliveries"
    },
    tempJson: [],
    jsonOnMap: [],
    isolationState: false
  }),

  methods: {
    onFileSelected(event) {
      const file = event.target.files[0];

      if (!file || file.type !== "application/json") {
        return alert("A JSON-type file must be selected");
      }

      // Credit: https://stackoverflow.com/a/754398/52160
      const reader = new FileReader();
      reader.readAsText(file, "UTF-8");

      reader.onload = evt => {
        const text = evt.target.result;
        try {
          this.jsonResult = JSON.parse(text);
          this.jsonToShow = this.jsonResult;
          this.jsonOnMap = this.jsonToShow;
        } catch (e) {
          alert("An error has been ocurred when loading the JSON");
        }
      };
      reader.onerror = evt => {
        console.error(evt);
      };
    },
    isolateRoute: function(markedRouted) {
      if (!this.isolationState) {
        // show selected route
        this.isolationState = true;
        console.log("Isolation state: " + this.isolationState);
        console.log(markedRouted.id);

        this.jsonOnMap = {
          routesList: []
        };

        var o = {
          id: markedRouted.origin.id,
          latitude: markedRouted.origin.latitude,
          longitude: markedRouted.origin.longitude,
          description: markedRouted.origin.description,
          address: markedRouted.origin.address
        };

        this.jsonOnMap.routesList.push({
          id: markedRouted.id,
          startDate: markedRouted.startDate,
          endDate: markedRouted.endDate,
          duration: markedRouted.duration,
          extraHours: markedRouted.extraHours,
          serviceLevel: markedRouted.serviceLevel,
          vehicle: markedRouted.vehicle,
          deliveries: markedRouted.deliveries,
          km: markedRouted.km,
          stopsList: [],
          origin: o,
          forTransit: markedRouted.forTransit,
          forAttention: markedRouted.forAttention,
          forWaitingTime: markedRouted.forWaitingTime,
          porLlenado: markedRouted.porLlenado,
          porLlenadoUnidades: markedRouted.porLlenadoUnidades,
          porLlenadoVolumen: markedRouted.porLlenadoVolumen,
          porLlenadoPeso: markedRouted.porLlenadoPeso,
          quantity: markedRouted.quantity,
          volume: markedRouted.volume,
          weight: markedRouted.weight
        });

        for (var j in markedRouted.stopsList) {
          var parada = markedRouted.stopsList[j];

          this.jsonOnMap.routesList[0].stopsList.push({
            id: parada.id,
            startDate: parada.startDate,
            endDate: parada.endDate,
            latitude: parada.latitude,
            longitude: parada.longitude,
            sequence: parada.sequence,
            deliveriesList: []
          });

          for (var k in parada.deliveriesList) {
            var entrega = parada.deliveriesList[k];

            this.jsonOnMap.routesList[0].stopsList[j].deliveriesList.push({
              id: entrega.id,
              description: entrega.description,
              sequence: entrega.sequence,
              address: entrega.address,
              clientId: entrega.clientId,
              clientDescription: entrega.clientDescription,
              startDate: entrega.startDate,
              endDate: entrega.endDate,
              waitingTime: entrega.waitingTime,
              km: entrega.km,
              transit: entrega.transit,
              latitude: entrega.latitude,
              longitude: entrega.longitude,
              stockiness: entrega.stockiness,
              quantity: entrega.quantity,
              volume: entrega.volume,
              weight: entrega.weight
            });
          }
        }
        console.log(this.jsonOnMap);
      }
    },
    showRoutesAgain: function() {
      //show all routes
      this.isolationState = false;
      console.log("Isolation state: " + this.isolationState);
      this.jsonOnMap = this.jsonToShow;
    },
    filterRoutesBasedOnDateSelection: function() {
      if (this.selectedDate === "All routes") {
        this.jsonToShow = this.jsonResult;
        this.jsonOnMap = this.jsonResult;
        this.isolationState = false;
        console.log("Isolation state: " + this.isolationState);
      } else {
        this.isolationState = false;
        console.log("Isolation state: " + this.isolationState);
        // first getting the filtered values into an array
        var filteredList = this.jsonResult.routesList.filter(
          route =>
            moment(route.startDate).format("DD-MM-YYYY") == this.selectedDate
        );
        //console.log(filteredList);

        //but as it's not in the format of the original json we have to convert it
        //but first, we'll create the structure of this json
        this.jsonToShow = {
          routesList: []
        };

        // now we'll put all of the data inside the json
        for (var i in filteredList) {
          var route = filteredList[i];
          var o = {
            id: route.origin.id,
            latitude: route.origin.latitude,
            longitude: route.origin.longitude,
            description: route.origin.description,
            address: route.origin.address
          };
          this.jsonToShow.routesList.push({
            id: route.id,
            startDate: route.startDate,
            endDate: route.endDate,
            duration: route.duration,
            extraHours: route.extraHours,
            serviceLevel: route.serviceLevel,
            vehicle: route.vehicle,
            deliveries: route.deliveries,
            km: route.km,
            stopsList: [],
            origin: o,
            forTransit: route.forTransit,
            forAttention: route.forAttention,
            forWaitingTime: route.forWaitingTime,
            porLlenado: route.porLlenado,
            porLlenadoUnidades: route.porLlenadoUnidades,
            porLlenadoVolumen: route.porLlenadoVolumen,
            porLlenadoPeso: route.porLlenadoPeso,
            quantity: route.quantity,
            volume: route.volume,
            weight: route.weight
          });
          for (var j in route.stopsList) {
            var parada = route.stopsList[j];

            this.jsonToShow.routesList[i].stopsList.push({
              id: parada.id,
              startDate: parada.startDate,
              endDate: parada.endDate,
              latitude: parada.latitude,
              longitude: parada.longitude,
              sequence: parada.sequence,
              deliveriesList: []
            });

            for (var k in parada.deliveriesList) {
              var entrega = parada.deliveriesList[k];

              this.jsonToShow.routesList[i].stopsList[j].deliveriesList.push({
                id: entrega.id,
                description: entrega.description,
                sequence: entrega.sequence,
                address: entrega.address,
                clientId: entrega.clientId,
                clientDescription: entrega.clientDescription,
                startDate: entrega.startDate,
                endDate: entrega.endDate,
                waitingTime: entrega.waitingTime,
                km: entrega.km,
                transit: entrega.transit,
                latitude: entrega.latitude,
                longitude: entrega.longitude,
                stockiness: entrega.stockiness,
                quantity: entrega.quantity,
                volume: entrega.volume,
                weight: entrega.weight
              });
            }
          }
        }

        this.jsonOnMap = this.jsonToShow;
      }
    },
    formatDate(date) {
      return moment(date).format("DD-MM-YYYY");
    },
    invertDeliveriesMode() {
      if (!this.seeDeliveries) {
        this.seeDeliveries = true;
        this.button.text = "See stops";
      } else {
        this.seeDeliveries = false;
        this.button.text = "See stops + deliveries";
      }
    }
  },

  computed: {
    jsonExists() {
      return this.jsonResult.length != 0;
    },
    getDates() {
      var datesList = [
        ...new Set(
          this.jsonResult.routesList
            .map(x => moment(x.startDate).format("DD-MM-YYYY"))
            .map(a => a)
        )
      ];
      datesList.unshift("All routes");
      return datesList;
    }
  },
  mounted() {
    this.windowHeight = window.innerHeight;
  }
};
</script>
