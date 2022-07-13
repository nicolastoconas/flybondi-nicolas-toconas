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
            :items="destinationsFullName"
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
            label="Destino (Opcional)"
            hide-details
            :items="destinationsFullName"
            prepend-icon="mdi-map"
            :item-disabled="disableItemDestination"
            single-line
            color="#f9ba15"
            :disabled="!flight.origin"
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
            color="#f9ba15"
            ><template v-slot:prepend>
              <v-icon color="#f9ba15"> mdi-currency-usd </v-icon>
            </template></v-select
          >
        </v-col>
      </v-row>

      <v-row class="mx-2 mt-5">
        <v-col cols="12" md="4">
          <v-slider
            :disabled="noLimit"
            class="mt-md-7 mt-5"
            color="#f9ba15"
            v-model="totalDays"
            label="Días de vacaciones"
            thumb-color="#f9ba15"
            thumb-label="always"
            track-color="gray"
          ></v-slider>
        </v-col>
        <v-col cols="12" md="2" class="pt-0">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <div v-bind="attrs" v-on="on">
                <v-switch
                  class="mt-md-9 mt-0 ml-md-4 ml-0"
                  v-model="noLimit"
                  label="Sin limite"
                  color="#f9ba15"
                ></v-switch>
              </div>
            </template>
            <span v-if="!noLimit"
              >Activar esta opción hará que se muestren todas las posibilidades
              de días de vacaciones en cada vuelo.
            </span>
            <span v-else
              >Desactivar esta opción filtrará los resultados en base a la
              cantidad de días de vacaciones que se deseen.</span
            >
          </v-tooltip>
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
                v-model="dateRangeText"
                label="Rango de fechas"
                prepend-icon="mdi-calendar"
                readonly
                v-bind="attrs"
                v-on="on"
                color="#f9ba15"
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
              color="#f9ba15"
            ></v-date-picker>
          </v-menu>
        </v-col>
        <v-col v-if="dates[0] && dates[1]" cols="12" md="2" class="pt-0">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <div v-bind="attrs" v-on="on">
                <v-switch
                  class="mt-md-9 mt-0 ml-md-4 ml-0"
                  v-model="exactDateSearch"
                  label="Fecha exacta"
                  color="#f9ba15"
                ></v-switch>
              </div>
            </template>
            <span v-if="!exactDateSearch"
              >Activar esta opción hará que se busque la fecha exacta de salida
              y retorno.
            </span>
            <span v-else
              >Desactivar esta opción mostrará los posibles vuelos dentro del
              rango de fechas establecido</span
            >
          </v-tooltip>
        </v-col>
      </v-row>
      <v-col cols="12" class="text-center"
        ><v-btn
          :disabled="!flight.price || !flight.origin"
          rounded
          large
          color="#f9ba15"
          @click="filterFlights"
          >Buscar vuelos</v-btn
        ></v-col
      >
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "SelectFlights",
  props: ["flights"],

  data: () => ({
    modal: true,
    flight: {
      origin: "",
      destination: "",
      price: "",
    },
    dates: ["2021-11-15", "2021-12-15"],
    menuDates: false,
    tableFlights: [],
    exactDateSearch: false,
    noLimit: false,
    totalDays: 14,
    disabled: false,
    destinationsFullName: [
      { value: "COR", name: "Cordoba" },
      { value: "MDZ", name: "Mendoza" },
      { value: "EPA", name: "Buenos Aires" },
      { value: "BRC", name: "Bariloche" },
    ],
    prices: [400, 600, 800],
  }),

  methods: {
    filterFlights() {
      this.tableFlights = []; //Reseteo de tabla

      // Separación entre vuelos de ida y vuelta
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

      //Funcion para buscar coincidencias en base a los criterios de filtrado
      const flightsFiltered = [];
      for (let i = 0; i < ida.length; i++) {
        for (let j = 0; j < vuelta.length; j++) {
          if (
            (ida[i].price + vuelta[j].price <= this.flight.price ||
              !this.flight.price) &&
            vuelta[j].data > ida[i].data &&
            ida[i].destination === vuelta[j].origin
          ) {
            flightsFiltered.push(ida[i], vuelta[j]);
          }
        }
      }

      //Se crea un nuevo array con datos de 2 vuelos, uno de ida y otro de vuelta
      const packDepartureReturn = [];
      for (let i = 0; i < flightsFiltered.length; i += 2) {
        packDepartureReturn.push(flightsFiltered.slice(i, i + 2));
      }

      //Se arma el objeto que se mostrará en la tabla en base a los dos pares de vuelos filtrados previamente
      packDepartureReturn.forEach((e) => {
        const fly = {
          totalPrice: `$ ${(e[0].price + e[1].price).toFixed(2)}`,
          priceDeparture: e[0].price,
          priceReturn: e[1].price,
          origin: this.checkName(e[0].origin),
          destination: this.checkName(e[0].destination),
          departureDate: e[0].data,
          returnDate: e[1].data,
          availabilityOnDeparture: e[0].availability,
          availabilityOnReturn: e[1].availability,
          vacationDays: this.daysOfVacations(e[1].data, e[0].data),
        };
        this.tableFlights.push(fly);
      });

      if (this.exactDateSearch) this.exactDate();
      if (!this.noLimit) this.exactDays();

      this.$emit("flightsFiltered", this.tableFlights, true);
    },
    exactDate() {
      const exact = this.tableFlights.filter((f) => {
        return (
          f.departureDate === this.dates[0] && f.returnDate === this.dates[1]
        );
      });
      this.tableFlights = exact;
    },
    exactDays() {
      const exactDays = this.tableFlights.filter((f) => {
        return f.vacationDays === this.totalDays || !this.totalDays;
      });
      this.tableFlights = exactDays;
    },
    disableItemOrigin(item) {
      if (
        item.value === this.flight.destination ||
        (item.value !== "COR" &&
          this.flight.destination === "MDZ" &&
          this.flight.destination !== "") ||
        (item.value !== "BRC" &&
          this.flight.destination === "EPA" &&
          this.flight.destination !== "") ||
        (item.value !== "COR" &&
          item.value !== "EPA" &&
          this.flight.destination === "BRC" &&
          this.flight.destination !== "") ||
        (item.value !== "BRC" &&
          item.value !== "MDZ" &&
          this.flight.destination === "COR" &&
          this.flight.destination !== "")
      ) {
        return true;
      }
    },
    disableItemDestination(item) {
      if (
        item.value === this.flight.origin ||
        (item.value !== "COR" &&
          this.flight.origin === "MDZ" &&
          this.flight.origin !== "") ||
        (item.value !== "BRC" &&
          this.flight.origin === "EPA" &&
          this.flight.origin !== "") ||
        (item.value !== "COR" &&
          item.value !== "EPA" &&
          this.flight.origin === "BRC" &&
          this.flight.origin !== "") ||
        (item.value !== "BRC" &&
          item.value !== "MDZ" &&
          this.flight.origin === "COR" &&
          this.flight.origin !== "")
      ) {
        return true;
      }
    },
    daysOfVacations(dateOne, dateTwo) {
      return Math.round(
        (new Date(dateOne).getTime() - new Date(dateTwo).getTime()) /
          (1000 * 60 * 60 * 24)
      );
    },
    checkName(name) {
      this.destinationsFullName.some((e) =>
        e.value === name ? (name = e.name) : ""
      );
      return name;
    },
  },

  computed: {
    dateRangeText() {
      return this.dates.join(" - ");
    },
  },

  watch: {
    dates() {
      if (this.dates[0] > this.dates[1]) {
        this.dates.sort();
      }
      const diff = this.daysOfVacations(this.dates[1], this.dates[0]);
      if (diff < this.totalDays) {
        this.totalDays = diff;
      }
    },

    exactDateSearch() {
      if (this.exactDateSearch === true) {
        this.noLimit = true;
      } else {
        this.noLimit = false;
      }
    },
  },
};
</script>
