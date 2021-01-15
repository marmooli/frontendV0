<template>
  <div>
    <head> </head>
    <div class="row">
      <div class="column">
        <p>Trans Form</p>
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
            <option v-for="item in send_curs" :value="item.val" :key="item.id">
              {{ item.val }}
            </option>
          </select>
          <button @click="toggle">T</button>
        </div>
        <p>
          - <b>{{ fee }}</b> {{ send_selected }} transaction fees
        </p>
        <p>
          = <b>{{ amountToEx }}</b> {{ send_selected }} will exchanged with
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
            min="0"
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
              v-for="item in receive_curs"
              :value="item.val"
              :key="item.id"
            >
              {{ item.val }}
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
      send_curs: {
        1: { id: 1, val: "EUR" },
        2: { id: 2, val: "USD" },
        3: { id: 3, val: "CAD" },
      },
      send_selected: "CAD",
      receive_curs: {
        1: { id: 1, val: "EUR", b_rate: 1, s_rate: 0.1 },
        2: { id: 2, val: "USD", b_rate: 2, s_rate: 0.2 },
        3: { id: 3, val: "CAD", b_rate: 3, s_rate: 0.3 },
      },
      receive_selected: "USD",
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
        // update receive
        setTimeout(() => {
          this.send_cur = Number((this.send_cur * 1).toFixed(2));
          this.receive_cur = Number(
            ((this.send_cur - this.fee) * this.rate).toFixed(2)
          );
          if (this.receive_cur < 0) {
            this.receive_cur = 0;
          }
        }, 500);
      } else {
        //update send
        setTimeout(() => {
          this.receive_cur = Number((this.receive_cur * 1).toFixed(2));
          this.send_cur = Number(
            (this.receive_cur / this.rate + this.fee).toFixed(2)
          );
        }, 500);
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