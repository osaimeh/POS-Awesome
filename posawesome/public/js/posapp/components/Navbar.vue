<template>
  <nav>
    <v-app-bar app height="40" class="elevation-2">
      <v-toolbar-title class="text-uppercase indigo--text">
        <span>Avvio</span>
        <span class="font-weight-light">POS</span>
      </v-toolbar-title>

      <v-spacer></v-spacer>
      <div class="text-center" v-if="prev_invoice">
        <span class="pink--text">{{ prev_invoice }}</span>
      </div>
      <v-spacer></v-spacer>
      <div class="text-center">
        <v-menu offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="grey" dark text v-bind="attrs" v-on="on"
              >القائمة</v-btn
            >
          </template>
          <v-card class="mx-auto" max-width="300" tile>
            <v-list dense>
              <v-list-item-group v-model="menu_item" color="primary">
                <v-list-item @click="close_shift_dialog">
                  <v-list-item-icon>
                    <v-icon>mdi-folder-open</v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>اغلاق الوردية</v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item @click="go_invoices">
                  <v-list-item-icon>
                    <v-icon>mdi-note-multiple</v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title>فواتير المبيعات</v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>
        </v-menu>
      </div>

      <div class="text-center">
        <v-menu offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="grey" dark text v-bind="attrs" v-on="on"
              >الصفحات</v-btn
            >
          </template>
          <v-card class="mx-auto" max-width="300" tile>
            <v-list dense>
              <v-list-item-group v-model="item" color="primary">
                <v-list-item
                  v-for="(item, index) in items"
                  :key="item.text"
                  @click="[changePage(item.text), (item = index)]"
                >
                  <v-list-item-icon>
                    <v-icon v-text="item.icon"></v-icon>
                  </v-list-item-icon>
                  <v-list-item-content>
                    <v-list-item-title v-text="item.text"></v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>
        </v-menu>
      </div>
      <v-btn text color="grey">
        <span right>{{ pos_profile.name }}</span>
      </v-btn>
      <v-btn text color="grey" @click="go_desk">
        <span right>الرئيسية</span>
      </v-btn>
      <v-btn text color="grey" @click="logOut">
        <span right>الخروج</span>
      </v-btn>
    </v-app-bar>

    <v-snackbar v-model="snack" :timeout="5000" :color="snackColor" top right>
      {{ snackText }}
      <!-- <template v-slot:action="{ attrs }"> -->
      <!-- <v-btn v-bind="attrs" text @click="snack = false">Close</v-btn> -->
      <!-- </template> -->
    </v-snackbar>
  </nav>
</template>

<script>
import { evntBus } from '../bus';

export default {
  // components: {MyPopup},
  data() {
    return {
      item: 0,
      items: [{ text: 'POS', icon: 'mdi-point-of-sale' }],
      page: '',
      fav: true,
      menu: false,
      message: false,
      hints: true,
      menu_item: 0,
      snack: false,
      snackColor: '',
      snackText: '',
      company: 'POS Awesome',
      company_img: '/assets/erpnext/images/erp-icon.svg',
      pos_profile: '',
      prev_invoice: '',
    };
  },
  methods: {
    changePage(key) {
      this.$emit('changePage', key);
    },
    go_desk() {
      frappe.set_route('');
      location.reload();
    },
    go_invoices() {
      const url = window.location.origin + '/desk#List/Sales Invoice/List';
      const win = window.open(url, '_blank');
      win.focus();
    },
    close_shift_dialog() {
      evntBus.$emit('open_closing_dialog');
    },
    show_mesage(data) {
      this.snack = true;
      this.snackColor = data.color;
      this.snackText = data.text;
    },
    logOut() {
      var me = this;
      me.logged_out = true;
      return frappe.call({
        method: 'logout',
        callback: function (r) {
          if (r.exc) {
            return;
          }
          frappe.set_route('/login');
          location.reload();
        },
      });
    },
  },
  created: function () {
    this.$nextTick(function () {
      evntBus.$on('show_mesage', (data) => {
        this.show_mesage(data);
      });
      evntBus.$on('set_company', (data) => {
        this.company = data.name;
        this.company_img = data.company_logo
          ? data.company_logo
          : this.company_img;
      });
      evntBus.$on('register_pos_profile', (data) => {
        this.pos_profile = data.pos_profile;
      });
      evntBus.$on('set_prev_invoice', (data) => {
        this.prev_invoice = data;
      });
    });
  },
};
</script>

<style scoped>
.margen-top {
  margin-top: 0px;
}
</style>
