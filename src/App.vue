<template>
  <v-app>
    <v-app-bar app color="white" dark>
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

      <v-btn href="#" target="_blank" text color="yellow">
        <span class="mr-2">Mis vuelos</span>
        <v-icon>mdi-airplane</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <SelectFlights
        :origins="origins"
        :destinations="destinations"
        :prices="prices"
        :flights="flights"
        @flightsFiltered="flightsFiltered"
      />
      <TableFlights
        :itemsTable="itemsTable"
        :headers="headers"
      />
    </v-main>
  </v-app>
</template>

<script>
import SelectFlights from "./components/SelectFlights";
import TableFlights from "./components/TableFlights";
import axios from "axios";

export default {
  name: "App",

  components: {
    SelectFlights,
    TableFlights,
  },

  data: () => ({
    flights: [],
    origins: [
      { name: "Cordoba", value: "COR" },
      { name: "Mendoza", value: "MDZ" },
      { name: "Bariloche", value: "BRC" },
      { name: "Buenos Aires", value: "EPA" },
    ],
    destinations: [
      { name: "Cordoba", value: "COR" },
      { name: "Mendoza", value: "MDZ" },
      { name: "Bariloche", value: "BRC" },
      { name: "Buenos Aires", value: "EPA" },
    ],
    prices: [400, 600, 800],
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
    ],
  }),

  created() {
    this.getDataFlights();
  },

  methods: {
    async getDataFlights() {
      const response = await axios.get("/flights.json");
      this.flights = response.data;
    },
    flightsFiltered(itemsTable) {
      this.itemsTable = itemsTable;
    },
  },
};
</script>
