<template>
  <v-container>
    <v-card elevation="8" shaped class="mt-5 mb-5">
      <v-row class="text-center mx-2">
        <v-col cols="6" md="4">
          <v-select
            v-model="flight.origin"
            item-text="name"
            item-value="value"
            menu-props="auto"
            label="Origen"
            hide-details
            :items="origins"
            :item-disabled="disableItemOrigin"
            single-line
            color="#f9ba15"
          >
            <template v-slot:prepend>
              <v-icon color="#f9ba15"> mdi-airplane-takeoff </v-icon>
            </template>
          </v-select>
        </v-col>
        <v-col cols="6" md="4">
          <v-select
            v-model="flight.destination"
            item-text="name"
            item-value="value"
            menu-props="auto"
            label="Destino"
            hide-details
            :items="destinations"
            prepend-icon="mdi-map"
            :item-disabled="disableItemDestination"
            single-line
            color="#f9ba15"
          >
            <template v-slot:prepend>
              <v-icon color="#f9ba15"> mdi-airplane-landing </v-icon>
            </template></v-select
          >
        </v-col>
        <v-col cols="12" md="4">
          <v-select
            v-model="flight.price"
            item-text="name"
            item-value="value"
            menu-props="auto"
            label="Precio"
            hide-details
            :items="prices"
            prepend-icon="mdi-map"
            single-line
            ><template v-slot:prepend>
              <v-icon color="#f9ba15"> mdi-currency-usd </v-icon>
            </template></v-select
          >
        </v-col>
      </v-row>

      <v-row class="mx-2 mt-5">
        <v-col cols="12" md="4">
          <div class="text-left"><strong> Días de vacaciones</strong></div>
          <v-radio-group class="mt-1" v-model="radio" row :disabled="disabled" >
            <v-radio color="#f9ba15" label="7 días" :value="7"></v-radio>
            <v-radio color="#f9ba15" label="14 días" :value="14"></v-radio>
            <v-radio color="#f9ba15" label="21 días" :value="21"></v-radio>
            <v-radio color="#f9ba15" label="Sin limite" value=""></v-radio>
          </v-radio-group>
        </v-col>
        <v-col cols="12" md="4">
          <v-menu
            v-model="menuDates"
            :close-on-content-click="false"
            :nudge-right="40"
            transition="scale-transition"
            offset-y
            min-width="auto"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="dates"
                label="Rango de fechas"
                prepend-icon="mdi-calendar"
                readonly
                v-bind="attrs"
                v-on="on"
              >
                <template v-slot:prepend>
                  <v-icon color="#f9ba15"> mdi-calendar</v-icon>
                </template></v-text-field
              >
            </template>
            <v-date-picker
              v-model="dates"
              range
              no-title
              @change="menuDates = false"
            ></v-date-picker>
          </v-menu>
        </v-col>
        <v-col v-if="dates[0] && dates[1]" cols="12" md="4" class="pt-0">
          <strong
            ><v-switch
              class="mt-md-9 mt-0 ml-md-4 ml-0"
              v-model="switch1"
              label="Fecha exacta"
              color="#f9ba15"
              @change="exactDate()"
            ></v-switch
          ></strong>
        </v-col>
      </v-row>
      <v-col cols="12" class="text-center"
        ><v-btn :disabled="!flight.price || !flight.origin" rounded large color="#f9ba15" @click="filterFlights"
          >Buscar vuelos!</v-btn
        ></v-col
      >
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "SelectFlights",
  props: ["origins", "destinations", "prices", "flights"],

  data: () => ({
    modal: true,
    flight: {
      origin: "",
      destination: "",
      price: "",
    },
    dates: [],
    menuDates: false,
    tableFlights: [],
    switch1: false,
    switch2: true,
    radio: 14,
    disabled: false,
  }),

  methods: {
    filterFlights() {
      this.tableFlights = [];
      const ida = this.flights.filter((flight) => {
        return (
          flight.origin === this.flight.origin &&
          (flight.destination === this.flight.destination ||
            !this.flight.destination) &&
          (flight.data >= this.dates[0] || !this.dates[0])
        );
      });
      const vuelta = this.flights.filter((flight) => {
        return (
          flight.destination === this.flight.origin &&
          (flight.origin === this.flight.destination ||
            !this.flight.destination) &&
          (flight.data <= this.dates[1] || !this.dates[1])
        );
      });
      const nuevoArray = [];
      for (let i in ida) {
        for (let j in vuelta) {
          if (
            (ida[i].price + vuelta[j].price < this.flight.price ||
              !this.flight.price) &&
            vuelta[j].data > ida[i].data &&
            ida[i].destination === vuelta[j].origin
          ) {
            nuevoArray.push(ida[i], vuelta[j]);
          }
        }
      }

      const vuelosFinales = [];
      for (let i = 0; i < nuevoArray.length; i += 2) {
        vuelosFinales.push(nuevoArray.slice(i, i + 2));
      }

      vuelosFinales.forEach((e) => {
        const fly = {
          totalPrice: e[0].price + e[1].price,
          priceDeparture: e[0].price,
          priceReturn: e[1].price,
          origin: e[0].origin,
          destination: e[0].destination,
          departureDate: e[0].data,
          returnDate: e[1].data,
          availabilityOnDeparture: e[0].availability,
          availabilityOnReturn: e[1].availability,
          vacationDays: this.daysOfVacations(e[1].data, e[0].data),
        };
        this.tableFlights.push(fly);
      });

      if (this.switch1) this.exactDate();
      if (this.radio) this.twoWeeks();
      this.modal = false;
      this.$emit("flightsFiltered", this.tableFlights);
    },
    exactDate() {
      const exact = this.tableFlights.filter((f) => {
        return (
          f.departureDate === this.dates[0] && f.returnDate === this.dates[1]
        );
      });
      this.tableFlights = exact;
    },

    twoWeeks() {
      const twoWeeks = this.tableFlights.filter((f) => {
        return f.vacationDays === this.radio || !this.radio;
      });
      this.tableFlights = twoWeeks;
    },
    disableItemOrigin(item) {
      if (item.value === this.flight.destination) {
        return true;
      }
    },
    disableItemDestination(item) {
      if (item.value === this.flight.origin) {
        return true;
      }
    },
    daysOfVacations(dateOne, dateTwo) {
      return Math.round(
        (new Date(dateOne).getTime() - new Date(dateTwo).getTime()) /
          (1000 * 60 * 60 * 24)
      );
    },
  },

  watch: {
    dates() {
      if (this.dates[0] > this.dates[1]) {
        this.dates.sort();
      }
    },
  },
};
</script>
