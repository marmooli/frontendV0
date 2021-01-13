<template>
  <div id="watch_vue">
    <p>{{ countDown }}</p>
    <p>
      <button @click="resetRate">update Rate</button>
    </p>
    <p>rate:{{ rate }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      countDown: 12,
      rate: 'updating...',
      flagReset: false,
      flagCount: true,
    };
  },
  mounted() {
    this.updateRate();
    this.countDownTimer();
  },
  watch: {
    flagCount: 'countDownTimer',
  },
  methods: {
    countDownTimer() {
      if (this.countDown < 1 || this.flagReset === true) {
        this.countDown = '-';
        this.rate = 'updating...';
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
      this.rate = 'updating...';
      this.flagReset = true;
    },
    updateRate() {
      const max = 100;
      const min = 1;
      const random = Math.floor(Math.random() * (max - min + 1) + min);
      this.rate = random;
    },
  },
};
</script>
