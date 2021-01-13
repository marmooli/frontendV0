<template>
  <!-- <HelloWorld msg="Hello Vue 3 in CodeSandbox!" /> -->
  <div>
    <!-- <Timer /> -->
    <TransferForm />
  </div>
</template>

<script>
// import HelloWorldVue from "./components/HelloWorld.vue";
// import Timer from "./components/Timer.vue";
import TransferForm from "./components/TransferForm";
export default {
  name: "App",
  components: {
    // HelloWorld: HelloWorldVue,
    // Timer: Timer,
    TransferForm: TransferForm,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
