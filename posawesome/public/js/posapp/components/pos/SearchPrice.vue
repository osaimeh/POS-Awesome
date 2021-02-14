<template>
  <v-row justify="center">
    <v-dialog v-model="Dialog" max-width="500">
      <v-card>
        <v-card-title>
          <span class="headline indigo--text">Search Item Price</span>
        </v-card-title>
        <v-card-text class="mt-2">
          <v-autocomplete
            dense
            clearable
            color="indigo"
            label="Item"
            v-model="item"
            :items="items"
            item-text="item_name"
            item-value="item_code"
            background-color="white"
            no-data-text="Item not found"
            hide-details
            :filter="customFilter"
          >
            <template v-slot:item="data">
              <template>
                <v-list-item-content>
                  <v-list-item-title
                    class="indigo--text subtitle-1"
                    v-html="data.item.item_name"
                  ></v-list-item-title>
                  <v-list-item-subtitle
                    v-html="`ID: ${data.item.item_code}`"
                  ></v-list-item-subtitle>
                  <v-list-item-subtitle
                    v-html="`Price: ${data.item.rate}`"
                  ></v-list-item-subtitle>
                </v-list-item-content>
              </template>
            </template>
          </v-autocomplete>
        </v-card-text>
        <v-card-text v-if="item_data" class="mb-0 pb-2">
          <v-row class="mb-0">
            <v-col cols="6">
              <h4>Item Name</h4>
              <h5>{{ item_data.item_name }}</h5>
            </v-col>
            <v-col cols="6">
              <h4>Item Code</h4>
              <h5>{{ item_data.item_code }}</h5>
            </v-col>
            <v-col cols="6">
              <h4>Item Price</h4>
              <h5>{{ formtCurrency(item_data.rate) }}</h5>
            </v-col>
            <v-col cols="6">
              <h4>Available QTY</h4>
              <h5>
                {{ formtCurrency(item_data.actual_qty) }}
                {{ item_data.stock_uom }}
              </h5>
            </v-col>
          </v-row>
        </v-card-text>

        <v-card-actions class="mt-0">
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  props: ['items'],
  data: () => ({
    Dialog: false,
    item: null,
    item_data: null,
  }),
  watch: {
    item() {
      const item_data = this.items.find((a) => a.item_code == this.item);
      this.item_data = item_data;
    },
  },
  methods: {
    close_dialog() {
      this.Dialog = false;
    },
    formtCurrency(value) {
      value = parseFloat(value);
      return value.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
    },
    customFilter(item, queryText, itemText) {
      const textOne = item.item_name ? item.item_name.toLowerCase() : '';
      const textTwo = item.item_code ? item.item_code.toLowerCase() : '';
      const textThree = [];
      item.item_barcode.forEach((element) => {
        textThree.push(element.barcode);
      });

      const searchText = queryText.toLowerCase();

      return (
        textOne.indexOf(searchText) > -1 ||
        textTwo.indexOf(searchText) > -1 ||
        textThree.includes(searchText)
      );
    },
  },

  mounted: function () {
    this.$nextTick(function () {});
  },

  created: function () {
    evntBus.$on('search_price', () => {
      this.Dialog = true;
    });
  },
};
</script>
