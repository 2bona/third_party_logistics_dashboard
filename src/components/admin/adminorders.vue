<template>
  <div class="grey lighten-4" style="width:100%; height: 100%; min-height:100vh">
    <v-row class="pa-3 mb-12 px-6">
      <v-flex xs12 class="mb-3 pt-2 px-0">
        <v-layout row wrap>
      

          <v-flex xs5 class="pl-3">
            <h1 class="title text--darken-2 font-weight-bold  mb-3 grey--text">
              Orders ({{orders.length || 0}}) <br/>
              <span
            v-if="dates.length"
            class="font-weight-bold body-1 text-capitalize"
            >({{
              !dates.length
                ? ""
                : dates.length > 1
                ? "" + sorted_dates[0] + " - " + sorted_dates[1]
                : "on " + sorted_dates[0]
            }})</span
          >
            </h1>
          </v-flex>
          <v-flex xs7 style="display: flex;
    justify-content: flex-end;">
             <v-btn 
             
               depressed color="grey lighten-2" rounded :loading="loadingHist" @click="showCalender()"   >
          <v-icon>mdi-history</v-icon><span>
            history
            </span> 
        </v-btn>
            <v-btn
            :loading="orderLoad"
              @click="start()" class="mx-3"
                depressed color="grey lighten-2" rounded
              ><v-icon>mdi-reload</v-icon>
              <span v-if=" $vuetify.breakpoint.smAndUp">
              reload
              </span>
              </v-btn
            >
          </v-flex>
        </v-layout>

        <v-card   style="overflow-x: scroll;border-radius: 25px" class="mt-4 grey lighten-4 pb-8">
          <v-card-title style="    position: sticky;
    left: 0;"> 
            <v-text-field
              v-model="search"
              dense style="position: sticky;
    left: 0px;"
              append-icon="mdi-magnify"
              label="Search by order id"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table
            class="grey lighten-4"
            :mobile-breakpoint="30"
            :headers="headers"
            v-model="selected"
            :items="orders"  
            style="min-width: 880px;"
            @click:row="clicker($event)"
            :expanded.sync="expanded"
            :search="search"
            disable-pagination
            hide-default-footer
          >
           <template v-slot:item.delivery.name="{ item }">
              <h2 v-if="item.delivery"
                class="title text-capitalize"
               :class="setColor(item.status)"
                >{{ item.delivery.name }}</h2
              >
            </template>
                  <template v-slot:item.address="{ item }">
              <span
              style="
    width: 188px!important;
    display: block;
"
              v-if="item.address"
                class=" font-weight-bold text-capitalize"
               :class="setColor(item.status)"
                >{{ (item.address.name) }}</span>
            </template>
           
            <template v-slot:item.status="{ item }">
              <h2
                class="title text-capitalize"
               :class="setColor(item.status)"
                v-text="
                  item.status === 1
                    ? 'In Processing'
                    : item.status === 2
                    ? 'Processed'
                    : item.status === 3
                    ? 'in-transit'
                    : item.status === 4
                    ? 'delivered'
                    : item.status === 5
                    ? 'rejected'
                    : 'unread'
                "
              ></h2>
            </template>
            <template v-slot:item.vendor.name="{ item }">
              <h2
                class="title text-capitalize"
               :class="setColor(item.status)"
                >{{ item.vendor.name }}</h2
              >
            </template>
            <template v-slot:item.created_at="{ item }">
              <h2
                class="title text-capitalize"
               :class="setColor(item.status)"
                >{{ item.created_at | nowDate }}</h2
              >
            </template>
            <template v-slot:item.id="{ item }">
              <h2
                class="title text-capitalize d-flex"
               :class="setColor(item.status)"
                ><v-icon
                  v-if="item.status === 3 || item.status === 1"
                  size="8"
                  class=" mr-2"
                  :color="item.status === 3 ? 'green' : 'orange'"
                  >mdi-circle</v-icon
                >{{ item.id }}</h2
              >
            </template>
            <template v-slot:item.payment_method="{ item }">
              <h2
                class=" title text-capitalize"
               :class="setColor(item.status)"
                >{{ paymentMethod(item.payment_method) }}</h2
              >
            </template>
            <template v-slot:item.tracking_id="{ item }">
              <h2
                class=" title text-capitalize"
               :class="setColor(item.status)"
                >{{ item.tracking_id }}</h2
              >
            </template>
            <template v-slot:item.delivery_fee="{ item }">
            <h2
                class=" title text-capitalize"
               :class="setColor(item.status)"
                ><v-icon 
                size="16" 
                
                class="pb-1"
                       :color="
                  item.status === 1
                    ? 'blue'
                    : item.status === 2
                    ? 'green'
                    : item.status === 3
                    ? 'orange'
                    : item.status === 4
                    ? 'grey'
                    : item.status === 5
                    ? 'red'
                    : ''
                ">mdi-currency-ngn</v-icon>{{ (item.type == 'Errands'? item.total - cut : item.delivery_fee - cut) | price }}</h2
              >
            </template>
            <template v-slot:expanded-item="{ headers, item }">
              <td :colspan="headers.length">{{ item }}</td>
            </template>
          </v-data-table>
          <v-progress-linear
            color="orange"
            v-show="loading"
            :indeterminate="loading"
          ></v-progress-linear>
        </v-card>
      </v-flex>
    </v-row>
    <div style="position:fixed;width:100%; top:0px">
      <v-progress-linear
        color="blue lighten-2" :height="'8px'"
        v-show="orderLoad"
        :indeterminate="orderLoad"
      ></v-progress-linear>
    </div>

        <v-dialog
      max-width="450"
      style="background:white;"
      background-color="white"
      class="white pa-"
      v-model="calender"
    >
      <v-row
        style="    height: 100%;
    margin: 0!important;
    background: #b7b7b7;"
        class="mx-auto py-3  "
      >
        <v-expand-transition>
          <v-col cols="12" class="d-flex" v-show="calender">
          <keep-alive>
            <v-date-picker
              v-model="dates"
              range
              class="mx-auto"
              color="orange darken-3"
            ></v-date-picker>
          </keep-alive>
          </v-col>
        </v-expand-transition>
        <v-col cols="12" v-if="calender" class="d-flex">
          <v-btn
            @click="calender = false"
            color="grey lighten-3"
            
            style="height:51px!important"
            rounded
            v-if="dates.length < 1 && calender"
            class="py-3 mx-auto "
            >close</v-btn
          >
          <v-btn
            @click="setDateAndCloseCalender()"
            style="height:51px!important"
            rounded
            v-if="dates.length > 0 && calender"
            class="mx-auto"
            >Load record <br />
            {{
              sorted_dates.length > 1
                ? "between " + sorted_dates[0] + " & " + sorted_dates[1]
                : "on " + sorted_dates[0]
            }}</v-btn
          >
        </v-col>
      </v-row>
    </v-dialog>
  </div>
</template>
<style>
.v-data-table > .v-data-table__wrapper > table > thead > tr > th, .v-data-table > .v-data-table__wrapper > table > tfoot > tr > th {
    font-size: 16px!important;
    text-transform: uppercase;
}
td h1.title{
  font-size:9px!important;
}
.v-data-table-header th.sortable:first-child{
      padding: 0px 0px 0px 14px!important;
}
.v-data-table-header th.sortable{
      padding: 0 5px!important;
}
</style>
<script>
import axios from "axios";


export default {

  data() {
    return {
      dates: [],
      minDate2: new Date(),
      minDate: new Date().toISOString(),
      dialogLoad: false,
      dialog: false,
      calender: false,
      dialog: false,
      orderLoad: false,
      content: "",
      expanded: [],
      loadingHist: false,
      dialog2: false,
      selected: [],
      minDate_: new Date(),
      pageClose: false,
      dialog3: false,
      valid: true,
      deleteId: "",
      intervalId: null,
      search: "",
      headers: [
        {
          text: "Trckn id",
          align: "center",
          align: "left",
          value: "tracking_id"
        },
        { text: "When  ", align: "center", value: "created_at" },
        { text: "Rider", align: "center",value: "delivery.name" },
        { text: "status", align: "center", value: "status" },
        { text: "To", align: "center",value: "address" },
        { text: "Vendor", align: "center", value: "vendor.name" },
        { text: "Delivery Fee", align: "center", value: "delivery_fee" },
      ],
      dialog4: false,
      rules: {
        required: value => !!value || "Required."
      },
      radios: "Thank you soo much, we will keep improving"
    };
  },
  computed: {
    orders() {
      return this.$store.getters.getAdminOrderList;
    },
    cut() {
      return this.$store.getters.getCut;
    },
     logistic_id() {
      return this.$store.getters.getLogisticId;
    },
       sorted_dates() {
      return this.dates.sort();
    },
    loading() {
      return this.$store.getters.getVendorLoadStatus;
    },
    vendor() {
      return this.$store.getters.getVendor;
    },
    user() {
      return this.$store.getters.getUser;
    },
    replys() {
      return this.$store.getters.getReplys;
    }
  },
  mounted() {
    const sn = this;
    sn.pageClose = true;
        this.orderLoad = true; 
    if(!sn.orders.length){
        sn.$store.dispatch("setAdminOrderList");
        this.orderLoad = false; 
    }else{
        this.orderLoad = false; 

    }
  },
  methods: {
        setDateAndCloseCalender() {
      this.calender = false;
      this.loadOrders();
    },
      loadOrders() {
      const sn = this;
      sn.orderLoad = true
      sn.loadingHist = true
        const when_date =
          this.sorted_dates.length > 1
              ? "?type=range&from=" +
                this.sorted_dates[0] +
                "&to=" +
                this.sorted_dates[1]
              : "?type=single&on=" + this.sorted_dates[0]
          var url = "/order/adminorderlist_third_party"+when_date+"&logistic_id="+this.logistic_id
   axios.get(url)
      .then((response)=>{
        sn.orderLoad = false
        sn.loadingHist = false
        this.$store.dispatch("setAdminDatedOrderList", response.data.orders)
        this.$store.dispatch("setCut", response.data.cut)
      }).catch((err)=>{
        sn.orderLoad = false
        sn.loadingHist = false
        console.log(err)
          sn.$store.dispatch("snack", {
            color: "green",
            text: "An error occured. Error : "+err
          });
      })
    },
    reload(){
        this.dates = []
      this.orderLoad = true
          var url = "/order/adminorderlist_third_party?type=single&on=" + new Date().toISOString()+"&logistic_id="+this.logistic_id
    axios.get(url)
      .then( (response) => {
      this.orderLoad = false
        this.$store.dispatch("setAdminDatedOrderList", response.data.orders)
        this.$store.dispatch("setCut", response.data.cut)
          this.$store.dispatch("snack", {
            color: "blue",
            text: "Orders Reloaded"
          });
      }).catch((err)=>{
              this.$store.dispatch("snack", {
            color: "green",
            text: "An error occured. Error : "+err
          });
              this.b(e.id)
        })
    },
    setColor(x){
      return  x === 1
                    ? 'blue--text'
                    : x === 2
                    ? 'green--text'
                    : x === 3
                    ? 'orange--text'
                    : x === 4
                    ? 'grey--text'
                    : x === 5
                    ? 'red--text'
                    : ''
    },
        showCalender() {
      this.dates = []
      this.calender = true;
    },

       paymentMethod(x) {
      let d = "";
      if (x === 1) {
        d = "App Paid";
      } else if (x === 2) {
        d = "Transfer";
      } else if (x === 4) {
        d = "Over the counter or use P.O.S";
      } else if (x === 6) {
        d = "App Paid";
      } else if (x === 3) {
          d = "Cash";
        }
      return d;
    },
      startCallBack: function(x) {
      console.log(x);
      this
    },
    endCallBack: function(x) {
      console.log(x);
    },
     checktimer(x, y) {
      return !x && !y;
    },
    start() {
      const sn = this;
     sn.reload()
    },
    navb() {
        if(!this.orders.length){
     sn.reload()
        }
    },
    clicker(e) {
    OrderSound.stop()
    if ( !this.orderLoad ) {
      this.orderLoad = true; 
      this.b(e.id)
    }
    },
    b(e){
          this.$store.dispatch("order", {
              id: e,
          action: null
        });
          }
  }
};
</script>
