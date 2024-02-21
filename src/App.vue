<template>
  <div class="main">
    <h1>Vue Calendar</h1>
    <h2>Date Picker - Range</h2>
    <div class="section">
      <DatePicker
          :date="[startDate, endDate]"
          auto-apply
          range
          @change-date="onChangeDate"
      />
    </div>
    <h2>Date Picker - Target</h2>
    <div class="section">
      <DatePicker
          :date="targetDate"
          auto-apply
          @change-date="onChangeDate"
      />
    </div>
  </div>
</template>

<script setup>
import {ref} from "vue";

import DatePicker from "./components/DatePicker.vue";

const startDate = ref('');
const endDate = ref('');
const targetDate = ref('');

const onChangeDate = (event) => {
  console.log('date', event);
}
</script>
<style scoped>

.section {
  display: flex;
  flex-direction: column;
  margin-bottom: 2em;
  background-color: #1b1c1e;
  border: 1px solid #7e7e7e;
  border-radius: 8px;
  padding: 16px;
  width: 40%;
  height: 380px;
}
</style>