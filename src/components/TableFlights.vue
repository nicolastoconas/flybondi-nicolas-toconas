<template>
  <v-container>
    <v-row class="text-center">
      <v-col>
        <v-data-table
          :headers="headers"
          :items="itemsTable"
          :items-per-page="15"
          @click:row="select"
          class="elevation-1"
          :sort-by="['departureDate', 'returnDate']"
          :sort-desc="[false, false]"
        ></v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "TableFlights",
  props: ["itemsTable", "headers"],

  data: () => ({}),
  methods: {
    select(item) {
      console.log(item);
    },
  },
};
</script>
