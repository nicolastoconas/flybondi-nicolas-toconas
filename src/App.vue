<template>
  <v-app>
    <v-app-bar app>
      <div class="d-flex align-center">
        <v-img
          alt="flybondi Logo"
          class="shrink mr-2"
          contain
          src="https://static.flybondi.com/f/77475/x/22b78b68f4/logo.svg"
          transition="scale-transition"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>
      <div class="mt-6">
        <v-switch
          v-model="$vuetify.theme.dark"
          inset
          label="Modo nocturno"
          persistent-hint
          color="#f9ba15"
        ></v-switch>
      </div>
    </v-app-bar>

    <v-main>
      <SelectFlights :flights="flights" @flightsFiltered="flightsFiltered" />
      <v-col v-if="loading" class="text-center">
        <v-progress-circular
          indeterminate
          color="#f9ba15"
        ></v-progress-circular>
      </v-col>
      <transition name="fade">
        <TableFlights
          v-if="searching && !loading"
          :itemsTable="itemsTable"
          :headers="headers"
          @dialog="dialog"
        />
      </transition>
      <ModalFlights v-if="show" @dialog="dialog" />
    </v-main>
  </v-app>
</template>

<script>
import SelectFlights from "./components/SelectFlights";
import TableFlights from "./components/TableFlights";
import ModalFlights from "./components/ModalFlights";
import axios from "axios";

export default {
  name: "App",

  components: {
    SelectFlights,
    TableFlights,
    ModalFlights,
  },

  data: () => ({
    flights: [],
    itemsTable: [],
    headers: [
      { text: "Origen", value: "origin" },
      { text: "Destino", value: "destination" },
      { text: "Precio (ida y vuelta)", value: "totalPrice" },
      { text: "Fecha de salida", value: "departureDate" },
      { text: "Fecha de regreso", value: "returnDate" },
      { text: "Asientos disponibles (IDA)", value: "availabilityOnDeparture" },
      { text: "Asientos disponibles (VUELTA)", value: "availabilityOnReturn" },
      { text: "Días de vacaciones", value: "vacationDays" },
      { text: "Detalles", value: "actions", sortable: false },
    ],
    show: false,
    loading: false,
    searching: false,
  }),

  created() {
    this.getDataFlights();
  },

  methods: {
    async getDataFlights() {
      try {
        const response = await axios.get("/flights.json");
        this.flights = response.data;
      } catch (err) {
        console.error(err);
      }
    },
    flightsFiltered(itemsTable, val) {
      this.loading = true;
      this.itemsTable = [];
      this.searching = val;
      setTimeout(() => {
        this.itemsTable = itemsTable;
        this.loading = false;
      }, 1000);
    },
    dialog(item) {
      this.show = item;
    },
  },
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
