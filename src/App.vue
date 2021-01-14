<template>
  <!-- <HelloWorld msg="Hello Vue 3 in CodeSandbox!" /> -->
  <div>
    <!-- <div class="row">
      <div class="column">
        <RatesTable />
      </div>
      <div class="column">
        <TransferForm />
      </div>
    </div>
    <div class="row">
    </div> -->
    <TransferForm />
  </div>
</template>

<script>
// import HelloWorldVue from "./components/HelloWorld.vue";
// import Timer from "./components/Timer.vue";
import TransferForm from "./components/TransferForm";
// import RatesTable from "./components/RatesTable";
// import BasicTable from "./components/BasicTable";
export default {
  name: "App",
  components: {
    // HelloWorld: HelloWorldVue,
    // Timer: Timer,
    TransferForm: TransferForm,
    // RatesTable: RatesTable,
    // BasicTable: BasicTable,
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
.row {
  display: flex;
}

.column {
  flex: 50%;
}
</style>
