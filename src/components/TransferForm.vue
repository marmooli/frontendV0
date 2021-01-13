<template>
  <div>
    <p>Trans F</p>
    <div>
      send:
      <input
        type="number"
        :disabled="validate ? '' : disabled"
        required="required"
        :min="fee"
        v-model="send_cur"
        @change="checkSend()"
      />
      <select name="send_cur" id="send_cur" v-model="send_selected">
        <option v-for="item in send_curs" :value="item.val" :key="item.id">
          {{ item.val }}
        </option>
      </select>
    </div>
    <p>- {{ fee }} trans. service f.</p>
    <p>= {{ amountToEx }} amount to ex</p>
    <p>rate:{{ rate }}</p>
    <p>time:{{ countDown }}</p>
    <div>
      receive:
      <input
        type="number"
        :disabled="validate ? '' : disabled"
        min="0"
        v-model="receive_cur"
        @change="checkReceive()"
      />
      <select name="receive_cur" id="receive_cur" v-model="receive_selected">
        <option v-for="item in receive_curs" :value="item.val" :key="item.id">
          {{ item.val }}
        </option>
      </select>
    </div>
    <p>
      <button @click="resetRate">update Rate</button>
    </p>
  </div>
</template>
  
<script>
export default {
  data() {
    return {
      send_cur: 100,
      receive_cur: 0,
      fee: 5.59,
      countDown: 12,
      rate: "updating...",
      send_curs: {
        1: { id: 1, val: "AAA" },
        2: { id: 2, val: "BBB" },
      },
      send_selected: "AAA",
      receive_curs: {
        1: { id: 1, val: "AAA" },
        2: { id: 2, val: "BBB" },
        3: { id: 3, val: "CCC" },
      },
      receive_selected: "CCC",
      flagReset: false,
      flagCount: true,
      flagSendChanged: false,
      flagReceiveChanged: false,
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
    send_cur: "updateReceive",
    receive_cur: "updateSend",
    send_selected: "resetRate",
    receive_selected: "resetRate",
    // send_cur: "checkSendCur",
    // receive_cur: "checkReceiveCur",
    // flagSendChanged: "updateReceive",
    // flagReceiveChanged: "updateSend",
  },

  methods: {
    countDownTimer() {
      if (this.countDown < 1 || this.flagReset === true) {
        // this.disabled = (this.disabled + 1) % 2;
        this.countDown = "-";
        this.rate = "updating...";
        setTimeout(() => {
          this.updateRate();
          this.countDown = 12 + 1;
          this.flagReset = false;
        }, 1000);
        // this.disabled = (this.disabled + 1) % 2;
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
      const max = 100;
      const min = 1;
      const random = Math.floor(Math.random() * (max - min + 1) + min);
      this.rate = random;
      // this.rate = 1;
      this.updateReceive();
    },
    checkReceiveCur() {
      this.flagSendChanged = true;
    },
    checkSend() {
      this.send_cur = Number((this.send_cur * 1).toFixed(2));
    },
    checkReceive() {
      this.receive_cur = Number((this.receive_cur * 1).toFixed(2));
    },
    updateReceive() {
      setTimeout(() => {
        this.receive_cur = Number(
          ((this.send_cur - this.fee) * this.rate).toFixed(2)
        );
        if (this.receive_cur < 0) {
          this.receive_cur = 0;
        }
        this.flagSendChanged = false;
      }, 500);
    },
    updateSend() {
      setTimeout(() => {
        this.send_cur = Number(
          (this.receive_cur / this.rate + this.fee).toFixed(2)
        );
        this.flagReceiveChanged = false;
      }, 500);
    },
  },
  computed: {
    validate() {
      return this.countDown === "-" || this.flagReset === true;
    },
    amountToEx() {
      return Number((this.send_cur - this.fee).toFixed(2));
    },
  },
};
</script>