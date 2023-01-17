<template>
  <div>
    <!-- Status Card -->
    <Transaction-status :cardData="cardData" />

    <client-only>
      <v-skeleton-loader
        v-if="loadingPage"
        class="mx-auto mt-3"
        type="card"
      ></v-skeleton-loader>
      <div v-else class="content-box border-b shadow-0 pa-3">
        <v-card class="shadow-0">
          <div class="d-flex-center justify-space-between mb-4">
            <p class="side-p">{{ $t("best_selling") }}</p>
            <div class="d-flex gap-2 align-center">
              <order-sort
                :reset="reset"
                @orderVal="orderVal"
                :ranking="ranking"
              />
              <v-btn color="primary pa-0 rest-btn" @click="refrechAsyncData()">
                <v-icon class="me-1"> mdi-restart </v-icon>
              </v-btn>
            </div>
          </div>
          <v-data-table
            mobile-breakpoint="0"
            :search="search"
            :headers="bestSellHeader"
            :items="bestSellitems"
            :items-per-page="5"
            :disable-sort="true"
            :footer-props="{
              showFirstLastPage: false,
              'items-per-page-text': '    ',
            }"
            class="shadow-0"
            item-key="id"
          >
            <template v-slot:[`item.product`]="{ item }">
              <div class="d-flex gap-2 pa-2 ps-0">
                <img
                  width="90px"
                  height="60px"
                  class="cover-top rounded border-b"
                  :src="item.storeImage"
                />
                <div>
                  <p>{{ item.storeName }}</p>
                  <!-- <div class="tx-gray">{{ item.product.Productdescrption }}</div> -->
                </div>
              </div>
            </template>

            <template v-slot:[`item.order`]="{ item }">
              <v-list-item-icon v-if="item.storeId == storeID" class="ps-1">
                <v-icon small>mdi-hands-pray </v-icon>
              </v-list-item-icon>
            </template>
            <template v-slot:[`item.rate`]="{ item }">
              <rating :readonly="true" :ratingStore="item.rating" />
            </template>
          </v-data-table>
        </v-card>
      </div>
    </client-only>
  </div>
</template>

<script>
export default {
  middleware: ["auth"],
  layout: "default",
  computed: {
    storeID() {
      return this.$store.state.storeId;
    },
  },
  data() {
    return {
      ranking: [],
      search: "",
      cardData: [],
      reset: false,
      notficationNum: null,
      storesID: null,
      bestSellHeader: [
        { text: this.$t("products"), value: "product" },
        { text: this.$t("number_of_sales"), value: "count" },
        { text: this.$t("order"), value: "order" },
        { text: this.$t("rate"), value: "rate" },
      ],
      bestSellitems: [],
    };
  },

  asyncData({ $axios }) {
    return $axios.$get("storeIndex").then((res) => {
      return {
        bestSellitems: res.data.stores,
        storesID: res.data.stores.storeId,
        cardData: res.data.steps,
        ranking: res.data.ranking,
        notficationNum: res.data.notifyCount,
        loadingPage: false,
        reset: false,
      };
    });
  },
  methods: {
    refrechAsyncData() {
      this.$nuxt.refresh();
      this.reset = true;
    },
    orderVal(orderVal) {
      this.loadingPage = true;
      this.$axios
        .$get("storeIndex", { params: { filter: orderVal } })
        .then((res) => {
          (this.orderItems = res.data), (this.loadingPage = false);
        });
    },
  },
};
</script>

<style lang="scss" scoped>
.v-application .elevation-1 {
  box-shadow: none;
}
.cover-top {
  object-fit: cover;
  object-position: top;
}
</style>
