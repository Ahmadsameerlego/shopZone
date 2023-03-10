<template>
  <div>
    <!-- Order Tabs -->
    <div class="overflow-x-auto content-box border-b shadow-0">
      <div
        id="tab-container"
        class="orders-type-conatiner scroll-responsive d-flex align-center justify-space-between gap-2"
      >
        <div
          @click="selectOrder('waitForAccept')"
          :class="{ active: active_el == 'waitForAccept' }"
          class="type-order align-center text-center"
        >
          <span class="mt-2">{{ $t("newOrder") }}</span>
          <span class="num mt-2">{{ orderWaitForAccept }}</span>
        </div>
        <div
          @click="selectOrder('processing')"
          :class="{ active: active_el == 'processing' }"
          class="type-order align-center text-center"
        >
          <span class="mt-2">{{ $t("orderProgress") }} </span>
          <span class="num mt-2">{{ orderProcessing }}</span>
        </div>
        <div
          @click="selectOrder('finishOrder')"
          :class="{ active: active_el == 'finishOrder' }"
          class="type-order align-center text-center"
        >
          <span class="mt-2">{{ $t("finishedOrder") }}</span>
          <span class="num mt-2">{{ orderFinish }}</span>
        </div>
        <div
          @click="selectOrder('returnOrder')"
          :class="{ active: active_el == 'returnOrder' }"
          class="type-order align-center text-center"
        >
          <span class="mt-2">{{ $t("returnOrder") }}</span>
          <span class="num mt-2">{{ returnOrder }}</span>
        </div>
      </div>
    </div>
    <v-skeleton-loader
      v-if="loadingPage"
      class="mx-auto mt-3"
      type="card"
    ></v-skeleton-loader>
    <div v-else class="order-page mt-4 content-box border-b shadow-0">
      <client-only>
        <v-card
          class="top-table align-center border--bottom gap-2 shadow-0 pa-1 d-flex flex-column flex-sm-row flex-wrap justify-space-between w-100 mb-1"
        >
          <div class="d-flex justify-center gap-2 mb-1 flex-wrap">
            <div class="search-tabel">
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                :label="$t('search_what_you_want')"
                single-line
                hide-details
              ></v-text-field>
            </div>
          </div>

          <div class="d-flex gap-2">
            <!-- :reset="reset" -->
            <order-sort @orderVal="orderVal" :ranking="ranking" />
          </div>
        </v-card>

        <v-data-table
          class="text-center shadow-0"
          mobile-breakpoint="0"
          :search="search"
          :headers="headers"
          :items="orderItems"
          :items-per-page="5"
          :footer-props="{
            showFirstLastPage: false,
            'items-per-page-text': '',
          }"
          @click:row="(item) => routeDetails(item.orderId)"
        >
        </v-data-table>
      </client-only>
    </div>
  </div>
</template>

<script>
export default {
  middleware: ["auth"],
  name: "DataTableActions",
  data() {
    return {
      search: "",
      ranking: [],
      orderStatus: null,
      orderWaitForAccept: null,
      orderProcessing: null,
      orderFinish: null,
      returnOrder: null,
      selectedState: null,
      active_el: null,
      loadingPage: true,
      headers: [
        {
          text: this.$t("orderNumber"),
          value: "orderNumber",
        },
        { text: this.$t("name_of_sender_request"), value: "user.userName" },
        { text: this.$t("orderDate"), value: "date" },
        { text: this.$t("orderTime"), value: "time" },
        { text: this.$t("orderPrice"), value: "totalPriceValue" },
      ],
      orderItems: [],
    };
  },

  mounted() {
    this.currentOrder();
  },

  methods: {
    routeDetails(orderID) {
      this.$router.push(`order/${orderID}`);
    },

    // get orders firt
    currentOrder() {
      localStorage.setItem("orderStatus", "waitForAccept");
      this.active_el = "waitForAccept";
      this.selectedState = "waitForAccept";

      this.$axios
        .$get("ordersStore", {
          params: { filter: "filter", orderStatus: "waitForAccept" },
        })
        .then((res) => {
          this.orderItems = res.data.orders;
          this.ranking = res.data.ranking;
          this.orderWaitForAccept = res.data.orderWaitForAccept;
          this.orderProcessing = res.data.orderProcessing;
          this.orderFinish = res.data.orderFinish;
          this.returnOrder = res.data.returnOrder;
          this.orderStatus = res.data.returnOrder;
          this.loadingPage = false;
        });
    },

    selectOrder(orderStatus) {
      localStorage.setItem("orderStatus", orderStatus);
      this.selectedState = orderStatus;
      this.active_el = orderStatus;
      this.loadingPage = true;
      this.$axios
        .$get("ordersStore", {
          params: { filter: "filter", orderStatus: orderStatus },
        })
        .then((res) => {
          this.orderItems = res.data.orders;
          this.loadingPage = false;
        });
    },

    // ----------------------------------
    orderVal(orderVal) {
      this.$axios
        .$get("ordersStore", {
          params: { filter: orderVal, orderStatus: this.selectedState },
        })
        .then((res) => {
          this.orderItems = res.data.orders;
          this.loadingPage = false;
          this.currentOrder();
        });
    },
  },
};
</script>

<style lang="scss">
.border--bottom {
  padding-bottom: 15px !important;
  border-bottom: 1px solid #f5f5f5a9 !important;
}
.orders-type-conatiner {
  min-width: 70 0px;
  width: 100%;
  overflow-x: auto;
  padding: 5px 10px;
  .type-order {
    font-size: 14px;
    color: #737373;
    border-bottom: 2.5px solid transparent;
    transition: all 0.2s ease-in;
    padding-bottom: 3px;
    cursor: pointer;

    &.active,
    &:hover {
      color: #1b7bb1;
      border-bottom: 2.5px solid #1b7bb1;
    }
    .num {
      background: #f5f5f5;
      display: inline-block;
      padding: 2px 7px;
      margin: 0 3px;
      font-size: 12px;
      border-radius: 8px;
    }
  }
}
.order-page {
  .v-data-table > .v-data-table__wrapper > table > thead > tr:last-child > th {
    text-align: center;
  }

  tr {
    cursor: pointer;
    &:hover {
      color: rgb(27, 122, 177);
    }
  }

  td {
    position: relative;
    // cursor: pointer;
    &:hover {
      &:last-child {
        color: rgb(27, 122, 177);
        &::before {
          color: rgb(27, 122, 177);
          cursor: pointer;
        }
      }
    }
    &:last-child::before {
      position: absolute;
      left: 10px;
      content: "\F0141";
      display: inline-block;
      font: normal normal normal 24px/1 "Material Design Icons";
      font-size: inherit;
      text-rendering: auto;
      line-height: inherit;
    }
  }
  .top-table {
    box-shadow: 1px 1px 1px #ccc;
    .v-text-field input {
      line-height: 27px;
      padding-bottom: 0px !important;
    }
    .v-input {
      max-width: 40%;
      background-color: #f5f5f5;
      padding: 0 !important;
      padding-inline-start: 6px !important;
      margin: 0;
      border-radius: 3px !important;
    }
    .v-input__icon {
      .v-icon {
        border-end-end-radius: 3px !important;
        border-start-end-radius: 3px !important;
        padding: 5px;
        background-color: rgb(27, 122, 177) !important;
        color: #fff !important;
        font-size: 19px;
        width: 104%;
      }
    }
    .v-text-field > .v-input__control > .v-input__slot:before {
      display: none;
    }
  }
}

.app-container[dir="ltr"] {
  .order-page {
    td {
      &:last-child::before {
        right: 10px;
        left: auto;
        transform: rotate(180deg);
      }
    }
  }
}
</style>
