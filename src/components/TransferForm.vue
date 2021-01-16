<template>
  <div>
    <head> </head>
    <div class="row">
      <div class="column">
        <p>Transaction Form</p>
        <div>
          <b v-if="!isLockedReceiveCur">Send:</b>
          <span v-else @click="isLockedReceiveCur = false">Send:</span>
          <input
            type="number"
            :disabled="validate ? '' : disabled"
            :min="fee"
            step="0.01"
            placeholder="enter send amount"
            v-model="send_cur"
            @click="isLockedReceiveCur = false"
            @select="isLockedReceiveCur = false"
          />
          <select
            name="send_cur"
            id="send_cur"
            v-model="send_selected"
            title="select your currency, wich you want to exchange"
          >
            <option
              v-for="item in currencies.send_currencies"
              :value="item.symbol"
              :key="item.symbol"
            >
              {{ item.symbol }}
            </option>
          </select>
          <button @click="toggle">T</button>
        </div>
        <p>
          <b>-{{ transactionFee }}</b> {{ send_selected }}
          <select
            name="transaction type"
            v-if="send_selected == 'USD'"
            v-model="usd_feeType_selected"
          >
            <option
              v-for="feeType in transactionFees"
              :value="feeType.val"
              :key="feeType.id"
            >
              {{ feeType.val }}
            </option>
          </select>
          <select>
            <option
              v-for="item in currencies.send_currencies.find(
                (s) => s.symbol == this.send_selected
              ).sending_ways"
              :key="item.way"
              :value="item"
            >
              {{ item.way }}
            </option>
          </select>
          <!-- <p v-for="item in currencies.send_currencies[0].sending_ways" :key="item.way">{{item.way}}</p> -->
          <!-- <p>{{currencies.send_currencies.find( s => s.symbol == this.send_selected ).sending_ways}}</p> -->
          <!-- <p v-if="currencies.send_currencies.find( s => s.symbol == "USD" ) ">!!!!</p> -->
          <select
            name="transaction type"
            v-if="send_selected == 'EUR'"
            v-model="eur_feeType_selected"
          >
            <option
              v-for="feeType in eurTransactionFees"
              :value="feeType.type"
              :key="feeType.id"
            >
              {{ feeType.type }}
            </option>
          </select>
          <span v-if="send_selected == 'CAD'">
            <select name="transaction type" v-model="cad_feeType_selected">
              <option
                v-for="feeType in cadTransactionFees"
                :value="feeType.type"
                :key="feeType.id"
              >
                {{ feeType.type }}
              </option>
            </select>
            <!-- {{feeType.transFee}} -->
          </span>
          transaction fees
        </p>
        <p>
          <b>={{ amountToEx }}</b> {{ send_selected }}, which we exchange with
        </p>
        <p>
          your guaranteed rate: <b>{{ rate }}</b>
        </p>
        <p>
          (will be updated in <b>{{ countDown }}</b> sec.)
        </p>
        <div>
          <b v-if="isLockedReceiveCur">Receive:</b>
          <span v-else @click="isLockedReceiveCur = true">Receive:</span>
          <input
            type="number"
            :disabled="validate ? '' : disabled"
            step=".01"
            placeholder="enter receive amount"
            v-model="receive_cur"
            @click="isLockedReceiveCur = true"
            @select="isLockedReceiveCur = true"
          />
          <select
            name="receive_cur"
            id="receive_cur"
            v-model="receive_selected"
          >
            <option
              v-for="item in currencies.receive_currencies"
              :value="item.symbol"
              :key="item.symbol"
            >
              {{ item.symbol }}
            </option>
          </select>
        </div>
        <p>
          <button title="With submiting you agree terms and conditions">
            Submit
          </button>
        </p>
      </div>
    </div>
  </div>
</template>
  
<script>
const axios = require("axios").default;
export default {
  data() {
    return {
      send_cur: 100,
      receive_cur: "",
      fee: 5.59,
      countDown: 12,
      rate: "updating...",
      currencies: {
        send_currencies: [
          {
            symbol: "USD",
            sending_ways: [
              { way: "usd_fast", fee: 5.99 },
              { way: "usd_middle", fee: 3.49 },
              { way: "usd_slow", fee: 1.85 },
            ],
          },
          {
            symbol: "EUR",
            sending_ways: [
              { way: "eur_fast", fee: 4.99 },
              { way: "eur_middle", fee: 2.49 },
              { way: "eur_slow", fee: 0.85 },
            ],
          },
        ],
        receive_currencies: [{ symbol: "USD" }, { symbol: "EUR" }],
      },
      send_selected: "EUR",
      receive_selected: "USD",
      transactionFees: {
        1: { id: 1, val: "fast", transFee: 5.99 },
        2: { id: 2, val: "middle", transFee: 3.49 },
        3: { id: 3, val: "slow", transFee: 1.85 },
      },
      eurTransactionFees: {
        1: { id: 1, type: "PayPal", transFee: 4.99 },
        2: { id: 2, type: "SEPA", transFee: 2.49 },
        3: { id: 3, type: "xyz", transFee: 0.85 },
      },
      cadTransactionFees: {
        1: { id: 1, type: "PayPal", transFee: 6.99 },
        2: { id: 2, type: "Wire", transFee: 4.49 },
        3: { id: 3, type: "Email", transFee: 2.85 },
        4: { id: 4, type: "Stripe", transFee: 1.45 },
      },
      usd_feeType_selected: "middle",
      eur_feeType_selected: "PayPal",
      cad_feeType_selected: "Wire",
      flagReset: false,
      flagCount: true,
      flagSendChanged: false,
      flagReceiveChanged: false,
      isLockedReceiveCur: false,
      disabled: 0,
      validated: 0,
    };
  },
  mounted() {
    this.updateRate();
    this.countDownTimer();
  },
  watch: {
    flagCount: "countDownTimer",
    send_cur() {
      // this.isLockedReceiveCur= false;
      this.updateSendReceive();
    },
    receive_cur() {
      // this.isLockedReceiveCur= true;
      this.updateSendReceive();
    },
    send_selected: "resetRate",
    receive_selected: "resetRate",
  },

  methods: {
    countDownTimer() {
      if (this.countDown < 1 || this.flagReset === true) {
        this.countDown = "-";
        this.rate = "updating...";
        setTimeout(() => {
          this.updateRate();
          this.countDown = 12 + 1;
          this.flagReset = false;
        }, 1000);
      }
      setTimeout(() => {
        this.countDown -= 1;
        this.flagCount = !this.flagCount;
      }, 1000);
    },

    resetRate() {
      this.rate = "updating...";
      this.flagReset = true;
    },

    updateRate() {
      const max = 0.01;
      const min = -0.01;
      var rand_offset = Number((Math.random() * (max - min) + min).toFixed(4));
      console.log(rand_offset);
      let symbols = this.receive_selected;
      let base = this.send_selected;
      this.getRateAPI(base, symbols, rand_offset);
      this.updateSendReceive();
    },

    getRateAPI(base, symbols, rand_offset) {
      if (base === symbols) {
        this.rate = 1;
      } else {
        axios
          .get("https://api.ratesapi.io/api/latest", {
            params: {
              base: base,
              symbols: symbols,
            },
          })
          .then((response) => {
            this.rate = Number(
              (Object.values(response.data.rates)[0] + rand_offset).toFixed(4)
            );
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },

    updateSendReceive() {
      if (!this.isLockedReceiveCur) {
        // send locked
        if (this.send_cur !== "") {
          // update receive
          setTimeout(() => {
            if (this.send_cur < 0) {
              this.send_cur = "";
            }
            this.send_cur = Number((this.send_cur * 1).toFixed(2));
            this.receive_cur = Number(
              ((this.send_cur - this.fee) * this.rate).toFixed(2)
            );
            if (this.receive_cur < 0) {
              this.receive_cur = 0;
            }
          }, 0);
        }
      } else {
        if (this.receive_cur !== "") {
          //update send
          setTimeout(() => {
            if (this.receive_cur < 0) {
              this.receive_cur = "";
            }
            this.receive_cur = Number((this.receive_cur * 1).toFixed(2));
            this.send_cur = Number(
              (this.receive_cur / this.rate + this.fee).toFixed(2)
            );
          }, 0);
        }
      }
    },

    toggle() {
      let buffer = this.send_selected;
      this.send_selected = this.receive_selected;
      this.receive_selected = buffer;
    },
  },
  computed: {
    validate() {
      return this.countDown === "-" || this.flagReset === true;
    },

    amountToEx() {
      let amountToEx = Number((this.send_cur - this.fee).toFixed(2));
      if (amountToEx < 0) {
        amountToEx = 0;
      }
      return amountToEx;
    },

    lockOrUnlock() {
      let text = "U";
      if (this.isLockedReceiveCur) {
        text = "L";
      }
      return text;
    },

    transactionFee() {
      return this.fee;
    },
  },
};
</script>

<style>
.row {
  display: flex;
  margin: auto;
}

.column {
  flex: 50%;
}
.ratesTable {
  width: 100%;
  /* border: 1px solid; */
}
td {
  border: 1px solid #dddddd;
}
</style>