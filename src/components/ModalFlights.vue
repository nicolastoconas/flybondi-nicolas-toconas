<template>
  <v-row justify="center">
    <v-dialog
      v-model="dialog"
      max-width="650"
      @keydown.esc="destroyDialog"
      @click:outside="destroyDialog"
    >
      <v-card>
        <v-img
          height="100px"
          src="https://static.flybondi.com/f/77475/x/22b78b68f4/logo.svg"
          contain
        >
        </v-img>
        <v-row class="px-0 mx-0">
          <v-col cols="12" md="6">
            <v-card-text>
              <div class="font-weight-bold ml-8 mt-7">
                Ida : {{ myFlight.departureDate }}
              </div>
              <v-timeline align-top dense>
                <v-timeline-item small color="green">
                  <div>
                    <div class="font-weight-normal">
                      Salida de <strong>{{ myFlight.origin }}</strong>
                    </div>
                    <div></div>
                  </div>
                </v-timeline-item>
                <v-timeline-item small color="green">
                  <div>
                    <div class="font-weight-normal">
                      Llegada a <strong>{{ myFlight.destination }}</strong>
                    </div>
                    <div></div>
                  </div>
                </v-timeline-item>
              </v-timeline>
            </v-card-text>
            <v-card-text>
              <div class="font-weight-bold ml-8 mt-2">
                Vuelta : {{ myFlight.returnDate }}
              </div>
              <v-timeline align-top dense>
                <v-timeline-item small color="green">
                  <div>
                    <div class="font-weight-normal">
                      Salida de <strong>{{ myFlight.destination }}</strong>
                    </div>
                    <div></div>
                  </div>
                </v-timeline-item>
                <v-timeline-item small color="green">
                  <div>
                    <div class="font-weight-normal">
                      Llegada a <strong>{{ myFlight.origin }}</strong>
                    </div>
                    <div></div>
                  </div>
                </v-timeline-item>
              </v-timeline>
            </v-card-text>
          </v-col>
          <v-col cols="12" md="6">
            <v-card-text>
              <div class="font-weight-bold ml-8 mt-md-7">
                <span class="text-decoration-underline">Costos</span>
              </div>
              <v-row class="ml-8 mt-2">
                <div>
                  <div>
                    Precio pasaje ida $ {{ `${myFlight.priceDeparture}` }}
                  </div>
                  <div>
                    Precio pasaje vuelta $ {{ `${myFlight.priceReturn}` }}
                  </div>
                  <hr />
                  <div>Precio final: {{ `${myFlight.totalPrice}` }}</div>
                </div>
              </v-row>
            </v-card-text>
            <v-card-text>
              <div class="font-weight-bold ml-8 mt-md-15">
                <span class="text-decoration-underline"
                  >Información adicional</span
                >
              </div>
              <v-row class="ml-8 mt-2">
                <div>
                  <div>
                    Asientos disponibles ida :
                    {{ `${myFlight.availabilityOnDeparture}` }}
                  </div>
                  <div>
                    Asientos disponibles vuelta :
                    {{ `${myFlight.availabilityOnReturn}` }}
                  </div>
                  <div>
                    Días totales de vacaciones: {{ `${myFlight.vacationDays}` }}
                  </div>
                </div>
              </v-row>
            </v-card-text>
          </v-col>
        </v-row>
        <v-col cols="12" class="text-center"
          ><v-btn @click="destroyDialog" class="mr-3" color="#f9ba15" rounded
            >Volver</v-btn
          >
        </v-col>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  data: () => ({
    dialog: true,
    myFlight: {},
  }),

  created() {
    this.myFlight = JSON.parse(localStorage.getItem("flight"));
  },

  methods: {
    destroyDialog() {
      this.$emit("dialog", false);
    },
  },
};
</script>
