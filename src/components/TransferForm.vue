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
          <b>-{{ sendingWayFee }}</b> {{ send_selected }}
          <select v-model="way_selected">
            <option
              v-for="item in currencies.send_currencies.find(
                (s) => s.symbol == this.send_selected
              ).sending_ways"
              :key="item.way"
            >
              {{ item.way }}
            </option>
          </select>
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
import axios from "axios";

export default {
  data() {
    return {
      send_cur: -100,
      receive_cur: 100,
      fee: 5.59,
      countDown: 600,
      rate: "updating...",
      currencies: {
        send_currencies: [
          {
            symbol: "USD",
            sending_ways: [
              { way: "usd_fast", fee: 3.3 },
              { way: "usd_middle", fee: 3.2 },
              { way: "usd_slow", fee: 3.1 },
            ],
          },
          {
            symbol: "EUR",
            sending_ways: [
              { way: "eur_fast", fee: 4.3 },
              { way: "eur_middle", fee: 4.2 },
              { way: "eur_slow", fee: 4.1 },
            ],
          },
          {
            symbol: "CAD",
            sending_ways: [
              { way: "cad_fast", fee: 2.3 },
              { way: "cad_middle", fee: 2.2 },
              { way: "cad_slow", fee: 2.1 },
            ],
          },
        ],
        receive_currencies: [
          { symbol: "USD" },
          { symbol: "EUR" },
          { symbol: "CAD" },
        ],
      },
      send_selected: "EUR",
      receive_selected: "USD",
      way_selected: "eur_middle", // you must unfotunatly hardcode this
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
    console.log("mounted");
    this.updateRate();
    this.countDownTimer();
  },

  watch: {
    flagCount: "countDownTimer",
    send_cur() {
      console.log("send_cur watched!");
      if (!this.isLockedReceiveCur) {
        console.log("forwarded to update");
        this.updateSendReceive();
      }
    },
    // rate() {
    //   console.log("rate watched!");
    //   this.updateSendReceive();
    // },
    receive_cur() {
      console.log("receive_cur watched!");
      if (this.isLockedReceiveCur) {
        console.log("forwarded to update");
        this.updateSendReceive();
      }
    },
    way_selected() {
      console.log("way_selected watched!");
      this.updateSendReceive();
    },
    send_selected() {
      this.resetRate();
      this.updateRate();
    },
    receive_selected() {
      this.resetRate();
      this.updateRate();
    },
  },

  methods: {
    countDownTimer() {
      if (this.countDown < 1 || this.flagReset === true) {
        this.countDown = "-";
        this.rate = "updating...";
        setTimeout(() => {
          this.updateRate();
          this.countDown = 600 + 1;
          this.flagReset = false;
        }, 0);
      }
      setTimeout(() => {
        this.countDown -= 1;
        this.flagCount = !this.flagCount;
      }, 1000);
    },

    resetRate() {
      console.log("resetRate()");
      this.rate = "updating...";
      this.way_selected = this.currencies.send_currencies.find(
        (s) => s.symbol === this.send_selected
      ).sending_ways[0].way;
      this.flagReset = true;
    },

    updateRate() {
      console.log("updateRate()");
      const max = 0.01;
      const min = -0.01;
      var rand_offset = Number((Math.random() * (max - min) + min).toFixed(4));
      console.log(rand_offset);
      let symbols = this.receive_selected;
      console.log("receive selected " + this.receive_selected);
      let base = this.send_selected;
      console.log("send selected " + this.send_selected);
      this.getRateAPI(base, symbols, rand_offset);
      this.updateSendReceive();
    },

    getRateAPI(base, symbols, rand_offset) {
      console.log("getRateAPI()" + base + " " + symbols);
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
            console.log("response " + response.data.rates[0]);
            console.log("rate in axios loop" + this.rate);
          })
          .catch((error) => {
            console.log(error);
          });
      }
      console.log("rate after else= " + this.rate);
    },

    updateSendReceive() {
      console.log(
        "updateSendReceive() with send_cur= " +
          this.send_cur +
          " and receive_cur= " +
          this.receive_cur
      );
      if (!this.isLockedReceiveCur) {
        // send locked
        if (this.send_cur !== "") {
          // update receive
          if (this.send_cur < 0) {
            this.send_cur = -this.send_cur;
            console.log("send updated to " + this.send_cur);
          }
          this.send_cur = Number((this.send_cur * 1).toFixed(2));
          let atx = this.amountToEx;
          console.log("atx= " + atx);
          this.receive_cur = Number((atx * this.rate).toFixed(2));
          console.log("Receive updated to " + this.receive_cur);
        } else {
          this.receive_cur = "";
        }
      } else {
        if (this.receive_cur !== "") {
          //update send
          if (this.receive_cur < 0) {
            this.receive_cur = -this.receive_cur;
          }
          this.receive_cur = Number((this.receive_cur * 1).toFixed(2));
          this.send_cur = Number(
            (this.receive_cur / this.rate + this.fee).toFixed(2)
          );
        } else {
          this.send_cur = "";
        }
        console.log("Send updated to " + this.send_cur);
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
      let amountToEx = Number((this.send_cur - this.sendingWayFee).toFixed(2));
      if (amountToEx < 0) {
        amountToEx = 0;
      }
      return amountToEx;
    },

    sendingWayFee() {
      let b = this.currencies.send_currencies.find(
        (s) => s.symbol === this.send_selected
      ).sending_ways;
      let a = b.find((t) => t.way === this.way_selected).fee;
      return a;
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