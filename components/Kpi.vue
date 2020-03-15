<template>
  <v-flex md3>
    <div>
      <v-expansion-panels focusable :key="i" v-for="(route, i) in jsonResult.routesList">
        <v-expansion-panel>
          <v-expansion-panel-header>{{ route.id }}</v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-card class="mx-auto" max-width="400" flat>
              <v-list-item two-line>
                <v-list-item-content>
                  <v-list-item-title class="headline">
                    Route KPIs
                    <v-btn class="mx-2" dark small color="primary" @click="isolateRoute(route)">
                      <v-icon dark>mdi-eye</v-icon>
                    </v-btn>
                  </v-list-item-title>
                  <v-list-item-subtitle>{{ formatDate(route.startDate) }}</v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>

              <v-card-text>
                <v-row align="center" class="mx-0">
                  <h5>Id:</h5>

                  <div class="grey--text ml-4">{{ route.id }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Start date:</h5>

                  <div class="grey--text ml-4">{{ formatDateTime(route.startDate) }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>End date:</h5>

                  <div class="grey--text ml-4">{{ formatDateTime(route.endDate) }}</div>
                </v-row>
                <v-row align="center" class="mx-0">
                  <h5>Duration:</h5>

                  <div class="grey--text ml-4">{{ route.duration }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Extra Hours:</h5>

                  <div class="grey--text ml-4">{{ route.extraHours }}</div>
                </v-row>
                <v-row align="center" class="mx-0">
                  <h5>Service level:</h5>

                  <div class="grey--text ml-4">{{ route.serviceLevel * 100 }} %</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Vehicle:</h5>

                  <div class="grey--text ml-4">{{ route.vehicle }}</div>
                </v-row>
                <v-row align="center" class="mx-0">
                  <h5>Deliveries:</h5>

                  <div class="grey--text ml-4">{{ route.deliveries }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Km:</h5>

                  <div class="grey--text ml-4">{{ route.km }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Origin:</h5>

                  <div class="grey--text ml-4">{{ route.origin.description }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Quantity:</h5>

                  <div class="grey--text ml-4">{{ route.quantity }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Volume:</h5>

                  <div class="grey--text ml-4">{{ route.volume }}</div>
                </v-row>

                <v-row align="center" class="mx-0">
                  <h5>Weight:</h5>

                  <div class="grey--text ml-4">{{ route.weight }}</div>
                </v-row>
              </v-card-text>
            </v-card>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
    </div>
  </v-flex>
</template>

<script>
var moment = require("moment");

export default {
  name: "Kpi",

  data() {
    return {
      paradas: [],
      fechas: []
    };
  },

  props: {
    jsonResult: null,
    deliveriesMode: true
  },

  methods: {
    formatDateTime(date) {
      return (
        moment(date).format("DD-MM-YYYY") +
        " a las " +
        moment(date).format("HH:MM:SS")
      );
    },
    formatDate(date) {
      return moment(date).format("DD-MM-YYYY");
    },
    isolateRoute: function(route) {
      this.$emit("isolate-route", route);
    }
  }
};
</script>
