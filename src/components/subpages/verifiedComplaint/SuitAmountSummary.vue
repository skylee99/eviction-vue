<template>
  <div class="mx-auto secondary--text">
    <v-btn
      color="info"
      icon
      class="float-right ma-8 btn--plain"
      @click="$emit('close:form')"
    >
      <v-icon>mdi-close</v-icon>
    </v-btn>

    <div
      style="width: 80%"
      class="pt-10 pb-4 mx-auto text-left"
    >
      <div class="custom-title pt-8 pb-4">
        Summary of Suit Amount
      </div>
      <div class="pb-6">
        Based on the information provided, these are the total amounts due, to
        be added to your suit and requested in the Verified Complaint. Review
        carefully and confirm each amount, edit or delete.
      </div>
    </div>
    <v-container
      v-for="item in items"
      :key="item.name"
      fluid
      class="pa-0 mb-4"
    >
      <v-row
        no-gutters
        justify-center
      >
        <v-col class="d-flex justify-center">
          <!-- <v-checkbox v-model="item.completed" large class="primary-checkbox" off-icon="mdi-checkbox-blank-circle-outline" on-icon="mdi-check-circle">
                  </v-checkbox> -->
          <checkbox
            v-model="item.completed"
            style="margin-top: 70px"
          />
        </v-col>
        <v-col cols="10">
          <v-container>
            <v-row>
              <div
                class="elevated-bar align-center d-flex"
                @click="openClose(item.name)"
              >
                <v-col cols="6">
                  <div class="text-left pl-8">
                    {{ item.name }}
                  </div>
                </v-col>
                <v-col cols="3">
                  <div class="text-right primary--text">
                    {{ convertToCurrency(item.total) }}
                  </div>
                </v-col>
                <v-col
                  style="height: 100%"
                  cols="3"
                  class="pa-0 d-flex justify-end"
                >
                  <div
                    class="elevated-bar-btn d-flex align-center justify-center"
                  >
                    <v-icon style="font-size: 75px; color: #93aebf">
                      mdi-menu-{{ direction(item.name) }}
                    </v-icon>
                  </div>
                </v-col>
              </div>
            </v-row>
            <span v-if="item.name === open">
              <v-row class="mt-4">
                <v-col cols="6">
                  <div
                    class="
                      text-left
                      pl-8
                      font-weight-medium
                      info--text
                      text-uppercase
                    "
                    style="font-size: 0.9rem"
                  >
                    INCLUDES
                  </div>
                </v-col>
              </v-row>
              <v-row
                v-for="row in item.amounts"
                :key="row.dates"
              >
                <v-col
                  cols="6"
                  class="py-1"
                >
                  <div
                    class="text-left pl-8"
                    style="font-size: 0.9rem"
                  >
                    {{
                      item.name === "Past Due Rent"
                        ? row.timePeriodDisplay
                        : row.type
                          ? row.type
                          : row.title
                    }}
                  </div>
                </v-col>
                <v-col
                  cols="3"
                  class="py-1"
                >
                  <div
                    class="text-right font-weight-medium"
                    style="font-size: 0.9rem"
                  >
                    {{ convertToCurrency(row.amount) }}
                  </div>
                </v-col>
                <v-col />
              </v-row>
            </span>
          </v-container>
        </v-col>
        <v-col class="d-flex justify-center">
          <v-menu
            offset-y
            bottom
            left
            open-on-hover
          >
            <template v-slot:activator="{ on }">
              <v-btn
                style="margin-top: 70px"
                small
                icon
                color="primary"
                class="btn--plain"
                v-on="on"
              >
                <v-icon>mdi-dots-horizontal</v-icon>
              </v-btn>
            </template>

            <v-list dense>
              <v-list-item @click="view(item.id)">
                <v-list-item-title class="overline info--text">
                  <!-- <i
                            style="font-size: 20px"
                            :class="`icofont-search-document pr-4`"
                          ></i> -->
                  <v-icon
                    color="info"
                    class="pr-4"
                  >
                    $pencil
                  </v-icon>
                  EDIT
                </v-list-item-title>
              </v-list-item>

              <v-list-item @click="deleteUpload(item.id)">
                <v-list-item-title class="overline info--text">
                  <i
                    style="font-size: 20px"
                    :class="`icofont-close-circled pr-4`"
                  />
                  DELETE
                </v-list-item-title>
              </v-list-item>
              <!-- <v-list-item
                        v-for="(btn, i) in btns"
                        :key="i"
                        @click="btn.onclick"
                      >
                        <v-list-item-title class="overline info--text"
                          ><i
                            style="font-size: 20px"
                            :class="`icofont-${btn.icon} pr-4`"
                          ></i>
                          {{ btn.title }}</v-list-item-title
                        >
                      </v-list-item> -->
            </v-list>
          </v-menu>
        </v-col>
      </v-row>
    </v-container>
    <v-card-actions
      style="background-color: #fafbfc; border-radius: 0px 0px 10px 10px"
      class="justify-end py-4 pr-12 mt-12"
    >
      <v-btn
        rounded
        color="accent_light"
        class="mb-2 mt-4 btn--plain capital--btn"
        text
        @click="$router.push({ name: 'vc-verification' })"
      >
        go back
      </v-btn>
      <v-btn
        rounded
        color="accent_light"
        class="px-8 mb-2 mt-4 white--text capital--btn"
        depressed
        @click="next"
      >
        continue
      </v-btn>
    </v-card-actions>
  </div>
</template>

<script>
import Checkbox from "../../Checkbox.vue";
export default {
  name: "SuitAmountSummary",
  components: {
    Checkbox,
  },
  props: {
    pastDueRent: {
      type: Array,
    },
    lateCharges: Array,
    utilities: Array,
    fees: Array,
    otherCharges: Array,
    mileagePrice: String,
    property: Object,
    legalFees: Array,
    legalFeesPermitted: Boolean,
  },
  data() {
    return {
      items: [
        {
          name: "Past Due Rent",
          total: this.calculateTotal(this.pastDueRent),
          amounts: this.pastDueRent,
        },
        {
          name: "Additional Charges",
          total: this.calculateAdditionalChargeTotal(),
          amounts: [
            ...this.groupLateCharges(),
            ...this.utilities,
            ...this.fees,
            ...this.otherCharges,
          ],
        },
        {
          name: this.legalFeesPermitted ? "Legal Fees" : "Court Fees",
          total: this.calculateLegalTotal(),
          amounts: this.legalFees,
        },
      ],
      open: "",
    };
  },
  computed: {
    direction() {
      return (name) => (name === this.open ? "up" : "down");
    },
  },
  watch: {
    pastDueRent(newVal) {
      this.items[0].amounts = newVal;
      this.items[0].total = this.calculateTotal(newVal);
    },
    mileagePrice(newVal) {
      this.items[2].amounts[1].amount = newVal;
    },
  },
  activated() {
    window.scrollTo(0, 0);
  },
  methods: {
    next() {
      // this.$emit("update:data", { confirmation: this.statements });
      this.$emit("create:preview");
      this.$emit("update", {
        data: {
          // fees: [
          //   { amount: 30, type: "filing" },
          //   { amount: this.mileagePrice, type: "mileage" },
          // ],
        },
        steps: { suitSummary: "completed" },
      });
      this.$router.push({ name: "vc-filing" });
    },
    openClose(name) {
      if (this.open === name) {
        this.open = "";
      } else {
        this.open = name;
      }
    },
    calculateTotal(arr) {
      let total = 0;
      arr.forEach((item) => {
        if (item.amount) {
          total += parseFloat(item.amount);
        }
      });
      return total;
    },
    groupLateCharges() {
      const groupedItems = [];
      console.log(this.lateCharges);

      // Group the price items by type
      this.lateCharges.map((item) => {
        let existItem = groupedItems.find((group) => group.type === item.type);
        if (existItem) {
          existItem.amount =
            parseFloat(existItem.amount) + parseFloat(item.amount);
        } else {
          groupedItems.push(item);
        }
        return item;
      });
      return groupedItems;
    },
    calculateAdditionalChargeTotal() {
      console.log(
        this.calculateTotal(this.lateCharges) +
          this.calculateTotal(this.utilities) +
          this.calculateTotal(this.fees) +
          this.calculateTotal(this.otherCharges)
      );
      return (
        this.calculateTotal(this.lateCharges) +
        this.calculateTotal(this.utilities) +
        this.calculateTotal(this.fees) +
        this.calculateTotal(this.otherCharges)
      );
    },
    calculateLegalTotal() {
      // const propertyCity = this.$store.getters.allCitiesAndSubs.find(city => city.name == this.property.city);
      // return propertyCity ? propertyCity.mileagePrice : 0;
      return this.calculateTotal(this.legalFees);
    },
  },
};
</script>

<style>
</style>