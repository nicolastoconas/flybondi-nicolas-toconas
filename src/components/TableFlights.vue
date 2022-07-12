<template>
  <v-container>
    <v-row class="text-center">
      <v-col>
        <v-data-table
          :headers="headers"
          :items="itemsTable"
          :items-per-page="15"
          @click:row="select"
          class="elevation-4"
          :sort-by="['departureDate', 'returnDate']"
          :sort-desc="[false, false]"
          style="cursor:pointer"
        >
          <template v-slot:[`item.actions`]="{ item }">
            <v-btn color="#f9ba15" rounded x-small class="mr-2" @click="select(item)"> Ver detalle </v-btn>
          </template></v-data-table
        >
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
      localStorage.setItem("flight", JSON.stringify(item));
      this.$emit("dialog", true);
    },
  },
};
</script>
