<template>
  <div
      ref="inputRef"
      class="position-relative flex align-items-center"
      :style="{'width': width}"
  >
    <input
        v-model="inputValue"
        v-input-mounted
        class="date-picker-input form-input"
        :disabled="disabled"
        readonly
        style="cursor: pointer;"
        @focus="onFocusInput"
    >
  </div>
  <div
      v-if="isVisible"
      id="calendarContainer"
      ref="containerRef"
      v-container-mounted
      :style="{'left': `${pickerPos[0]}px`, 'top': `${pickerPos[1]}px`}"
      @wheel.prevent.stop
  >
    <div class="calendar-header w-100 pl-2 pr-2">
      <div
          class="calendar-left-arrow"
          @click="onClickPrevMonth"
      />
      <div
          class="flex justify-content-center w-100"
          @click="onClickToggleMode"
      >
        <span class="cursor-pointer">{{ formattedYear }}</span>
        <span class="cursor-pointer">{{ formattedMonth }}</span>
      </div>
      <div
          class="calendar-right-arrow"
          @click="onClickNextMonth"
      />
    </div>
    <div class="calendar-content border-top border-bottom">
      <div
          v-if="viewMode === 'date'"
          class="flex direction-column g-1 p-2 calendar-fade-in"
      >
        <div class="flex g-1">
          <div
              v-for="(weekday) of weekdayList()"
              :key="weekday"
              class="calendar-weekday"
          >
            {{ weekday }}
          </div>
        </div>
        <div
            v-for="(item) of weekendList"
            :key="item"
            class="flex g-1"
        >
          <div
              v-for="(dayItem) of item"
              :key="dayItem"
              class="calendar-date"
              :class="{
              'disabled': !dayItem.currentMonth,
              'section': dayItem.section,
              'selected': dayItem.selected
            }"
              @click="onClickDate(dayItem)"
          >
            {{ dayItem.day }}
          </div>
        </div>
      </div>
      <div
          v-else-if="viewMode === 'year-month'"
          class="flex calendar-fade-in w-100"
      >
        <div
            class="calendar-wheel flex direction-column align-items-center g-3 w-100 border-right"
            @wheel="onWheelYear"
        >
          <template
              v-for="(item, index) of 3"
              :key="item"
          >
            <span @click="onClickYear($event.target.textContent)">{{ currentYear + index - 1 }}</span>
          </template>
        </div>
        <div
            class="calendar-wheel flex direction-column align-items-center w-100 g-3"
            @wheel="onWheelMonth"
        >
          <template
              v-for="(item) of 3"
              :key="item"
          >
            <span @click="onClickMonth($event.target.textContent)">{{ MONTH_LIST[(currentMonth + item - 1) % 12] }}</span>
          </template>
        </div>
      </div>
    </div>
    <div
        v-if="viewMode === 'date'"
        class="flex justify-content-center align-items-center text-primary-default-color p-1 cursor-pointer w-100"
        @click="onClickToday"
    >
      오늘
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { onClickOutside } from '@vueuse/core';

const emits = defineEmits(['change-date', 'close-picker', 'update:modelValue']);
const props = defineProps({
  autoApply: {
    type: Boolean,
    default: false,
  },
  date: {
    type: [Date, String, Array],
    default: '',
  },
  modelValue: {
    type: [Date, String, Array],
    default: '',
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  range: {
    type: Boolean,
    default: false,
  },
  width: {
    type: String,
    default: '220px',
  },
});
const vInputMounted = {
  mounted: (el) => {
    if (props.range) el.value = props.date[0] ? `${formattedDate(props.date[0])} ~ ${formattedDate(props.date[1])}` : '';
    else el.value = props.date ? formattedDate(props.date) : '';
  },
};
const vContainerMounted = {
  mounted: (el) => {
    const inputEl = inputRef.value;
    const rect = inputRef.value.getBoundingClientRect();

    pickerPos.value[0] = inputEl.offsetLeft;
    pickerPos.value[1] = rect.y + rect.height + 4;

    if (pickerPos.value[1] + el.getBoundingClientRect().height > window.innerHeight) {
      pickerPos.value[1] = rect.y - el.getBoundingClientRect().height - 4;
    }
  },
};
const MONTH_LIST = [12, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
const formattedDate = (value) => {
  const date = new Date(value);

  return value ? `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}-${(date.getDate()).toString().padStart(2, '0')}` : '';
};

const containerRef = ref(null);
const currentMonth = ref(null);
const currentYear = ref(null);
const formattedMonth = ref(null);
const formattedYear = ref(null);
const endDate = ref(null);
const pickerPos = ref([0]);
const inputRef = ref(null);
const inputValue = ref(null);
const startDate = ref(null);
const targetDate = ref(null);
const weekendList = ref([]);
const viewMode = ref('date');
const isVisible = ref(false);

let $d = null;
let $Y = null;
let $M = null;
let $D = null;

const locale = 'ko';
const monthFormat = new Intl.DateTimeFormat(locale, { month: 'short' });
const yearFormat = new Intl.DateTimeFormat(locale, { year: 'numeric' });

const closePicker = () => {
  isVisible.value = false;
  emits('close-picker');
};
const createWeekendList = () => {
  const add = () => parse(new Date($Y, $M, $D + 1));
  const parse = (date) => {
    $d = new Date(date);
    $Y = $d.getFullYear();
    $M = $d.getMonth();
    $D = $d.getDate();
  };
  const temp = $d;
  parse($d);

  currentYear.value = $Y;
  currentMonth.value = $M;
  formattedMonth.value = monthFormat.format($d);
  formattedYear.value = yearFormat.format($d);

  const weekStart = 0;
  const firstDate = new Date($Y, $M, 1);
  const weekday = firstDate.getDay();
  const gap = (weekday < weekStart ? weekday + 7 : weekday) - weekStart;
  const startWeekDay = new Date($Y, $M, -gap);
  const endDay = new Date($Y, $M + 1, 1);
  const list = [];

  parse(startWeekDay);

  while ($d.getTime() < endDay.getTime()) {
    const week = Array(7).fill(0).map(() => {
      add();

      const date = formattedDate($d);
      return {
        currentMonth: $M === currentMonth.value,
        date: date,
        day: $D,
      };
    });
    list.push(week);
  }

  parse(temp);
  weekendList.value = list;
};
const onClickDate = (target) => {
  if (props.range) {
    if (endDate.value) {
      endDate.value = null;
      startDate.value = target.date;
    } else if (startDate.value) {
      endDate.value = target.date;
    } else startDate.value = target.date;
  } else {
    targetDate.value = target.date;
  }

  if (!target.currentMonth) {
    $d = target.date;
    createWeekendList();
  }

  setWeekendList();

  if (props.range) {
    emits('change-date', {
      startDate: startDate.value,
      endDate: endDate.value,
    });
    if (props.autoApply && startDate.value && endDate.value) closePicker();
  } else {
    emits('change-date', targetDate.value);
    emits('update:modelValue', targetDate.value);
    if (props.autoApply) closePicker();
  }
};
const onClickMonth = (index) => {
  $d = new Date($Y, index - 1, 1);
  createWeekendList();
  onClickToggleMode();
};
const onClickNextMonth = () => {
  $d = new Date($Y, $M + 1, 1);
  createWeekendList();
};
const onClickPrevMonth = () => {
  $d = new Date($Y, $M - 1, 1);
  createWeekendList();
};
const onClickToday = () => {
  $d = formattedDate(new Date());

  createWeekendList();
  setWeekendList();
};
const onClickToggleMode = () => {
  viewMode.value = viewMode.value === 'date' ? 'year-month' : 'date';
  if (viewMode.value === 'date') {
    $d = new Date(currentYear.value, currentMonth.value, 1);
    createWeekendList();
  }
};
const onClickYear = (index) => {
  $d = new Date(index, $M, 1);
  createWeekendList();
  onClickToggleMode();
};
const onFocusInput = () => {
  isVisible.value = true;
};
const onWatchDate = (date) => {
  if (props.range) {
    startDate.value = date[0];
    endDate.value = date[1];
  } else {
    targetDate.value = formattedDate(date);

    $d = new Date(targetDate.value || new Date());
  }

  createWeekendList();
  setWeekendList();
};
const onWheelMonth = (event) => {
  if (event.deltaY > 0) currentMonth.value++;
  else currentMonth.value--;
  if (currentMonth.value > 11) currentMonth.value = 0;
  else if (currentMonth.value < 0) currentMonth.value = 11;
};
const onWheelYear = (event) => {
  if (event.deltaY > 0) currentYear.value++;
  else currentYear.value--;
};
const setWeekendList = () => {
  let dateList = null;

  if (props.range && endDate.value) {
    dateList = [new Date(startDate.value), new Date(endDate.value)];
    dateList.sort((a, b) => {
      return a > b ? 1 : -1;
    });
    startDate.value = formattedDate(dateList[0]);
    endDate.value = formattedDate(dateList[1]);
  }

  weekendList.value.forEach((weekend) => {
    weekend.forEach((item) => {
      const date = new Date(item.date);

      if (props.range) {
        item.section = dateList ? dateList[0] < date && date < dateList[1] : false;
        item.selected = startDate.value === item.date || endDate.value === item.date;
      } else item.selected = targetDate.value === item.date;
    });
  });

  if (props.range) inputValue.value = startDate.value ? `${startDate.value || ''}  ~  ${endDate.value || ''}` : '';
  else inputValue.value = `${targetDate.value}`;
};
const weekdayList = () => {
  const dateTimeFormat = new Intl.DateTimeFormat(locale, { weekday: 'short' });

  return Array(7).fill(0).map((item, index) => {
    return dateTimeFormat.format(new Date(1970, 1, index + 1));
  });
};

onClickOutside(containerRef, (event) => {
  if (event.target !== inputRef.value.children[0]) closePicker();
});

watch(() => props.date, (date) => {
  onWatchDate(date);
});
watch(() => props.modelValue, (date) => {
  onWatchDate(date);
});

onMounted(() => {
  if (props.range) {
    startDate.value = formattedDate(props.date[0] || '');
    endDate.value = formattedDate(props.date[1] || '');

    $d = new Date(endDate.value || new Date());
  } else {
    targetDate.value = formattedDate(props.date || props.modelValue);

    $d = new Date(targetDate.value || new Date());
  }

  createWeekendList();
  setWeekendList();
});
</script>
