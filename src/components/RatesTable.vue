<template>
  <div>
    <p>R T</p>
    <div>
      <p>
        base:
        <select name="receive_cur" id="receive_cur" v-model="receive_selected">
          <option v-for="item in receive_curs" :value="item.val" :key="item.id">
            {{ item.val }}
          </option>
        </select>
      </p>
    </div>

    <table class="ratesTable">
      <tr>
        <th>cur</th>
        <th>y b</th>
        <th>y s</th>
      </tr>
      <tr v-for="item in receive_curs" :key="item.id">
        <td>{{ item.val }}/{{ receive_selected }}</td>
        <td>
          {{ item.b_rate }}
        </td>
        <td>
          {{ item.s_rate }}
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      receive_curs: {
        1: { id: 1, val: "AAA", b_rate: 1, s_rate: 0.1 },
        2: { id: 2, val: "BBB", b_rate: 2, s_rate: 0.2 },
        3: { id: 3, val: "CCC", b_rate: 3, s_rate: 0.3 },
      },
      receive_selected: "CCC",
    };
  },
};
</script>

<style>
.ratesTable {
  width: 100%;
  /* border: 1px solid; */
}
td {
  border: 1px solid #dddddd;
}
</style>