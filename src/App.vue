<template>
  <div class="main">
    <div class="section">
      <h2>Date Picker</h2>
      <DatePicker
          :date="targetDate"
          auto-apply
          @change-date="onChangeDate"
      />
    </div>
    <div class="section">
      <h2>Date Picker - Range</h2>
      <DatePicker
          :date="[startDate, endDate]"
          auto-apply
          range
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
h2 {
  margin-bottom: 16px;
}
.section {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20%;
}
</style>