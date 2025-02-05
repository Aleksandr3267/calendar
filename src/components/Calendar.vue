<template>
  <div class="calendar-container bg-white rounded-lg shadow-md p-4">
    <div class="header flex justify-between items-center mb-4">
      <button @click="prevMonth" class="btn-nav-month text-gray-500 hover:text-gray-700">
        <svg width="24.000000" height="24.000000" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
          <g clip-path="url(#clip8_673)">
            <path id="Icon" d="M19 12L5 12M12 19L5 12L12 5" stroke="#0A0C13" stroke-opacity="1.000000" stroke-width="2.000000" stroke-linejoin="round" stroke-linecap="round"/>
          </g>
        </svg>
      </button>
      <span class="month-year text-lg font-medium">{{ currentMonthYear }}</span>
      <button @click="nextMonth" class="btn-nav-month text-gray-500 hover:text-gray-700">
        <svg width="24.000000" height="24.000000" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
          <g clip-path="url(#clip8_676)">
            <path id="Icon" d="M5 12L19 12M12 5L19 12L12 19" stroke="#0A0C13" stroke-opacity="1.000000" stroke-width="2.000000" stroke-linejoin="round" stroke-linecap="round"/>
          </g>
        </svg>
      </button>
    </div>
    <div class="grid grid-cols-7 gap-1">
      <div v-for="(day, index) in daysOfWeek" :key="day" :class="{'week-margin text-center font-medium': true, 'text-red-600': index === 6}">
        {{ day }}
      </div>
      <div v-for="date in dates" :key="date.date"
           :class="[ 
             'point-day text-center p-2', 
             date.isCurrentMonth ? '' : 'text-gray-400', 
             isSelected(date) ? '' : '', 
             isRange(date) ? '' : '', 
             isStart(date) ? 'rounded-full' : '', 
             isEnd(date) ? 'rounded-full' : '' 
           ]"
           :style="{
             backgroundColor: isSelected(date) || isStart(date) || isEnd(date) ? 'rgb(17, 49, 71)' : 
                              isRange(date) ? 'rgb(233, 234, 249)' : '',
             color: isSelected(date) || isStart(date) || isEnd(date) ? 'white' : 
                    isRange(date) ? 'rgb(10, 12, 19)' : 
                    (isSunday(date) && date.isCurrentMonth ? 'rgb(193, 16, 48)' : '') 
           }"
           @click="selectDate(date)"
      >
        {{ date.day }}
      </div>
    </div>
    <div class="button-container mt-4 text-center">
      <button @click="saveSelection" class="btn-gotovo">
        Готово
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import dayjs from 'dayjs';
import 'dayjs/locale/ru'; 

dayjs.locale('ru'); 

export default {
  props: {
    startDate: String,
    endDate: String,
  },
  emits: ['update-range'],
  setup(props, { emit }) {
    const currentMonth = ref(dayjs());
    const start = ref(props.startDate ? dayjs(props.startDate) : null);
    const end = ref(props.endDate ? dayjs(props.endDate) : null);
    const currentMonthYear = computed(() => currentMonth.value.format('MMMM YYYY'));
    const daysOfWeek = computed(() => ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']);
    const dates = computed(() => {
      const startOfMonth = currentMonth.value.startOf('month');
      const endOfMonth = currentMonth.value.endOf('month');
      const days = [];
      let currentDay = startOfMonth.startOf('week'); 
      const totalCells = 42; 

      while (days.length < totalCells) {
        days.push({
          date: currentDay.format('YYYY-MM-DD'),
          day: currentDay.date(),
          isCurrentMonth: currentDay.isSame(currentMonth.value, 'month'),
        });
        currentDay = currentDay.add(1, 'day');
      }
      return days;
    });


    const selectDate = (dateObj) => {
      const date = dayjs(dateObj.date);
      if (!start.value || (start.value && end.value)) {
        start.value = date;
        end.value = null;
      } else if (!end.value && date.isAfter(start.value, 'day')) {
        end.value = date;
      } else {
        start.value = date;
        end.value = null;
      }
    };

    const isSelected = (dateObj) => {
      const date = dayjs(dateObj.date);
      return start.value && date.isSame(start.value, 'day') || end.value && date.isSame(end.value, 'day');
    };

    const isRange = (dateObj) => {
      const date = dayjs(dateObj.date);
      return start.value && end.value && date.isAfter(start.value, 'day') && date.isBefore(end.value, 'day');
    };
    const isStart = (dateObj) => {
      const date = dayjs(dateObj.date);
      return start.value && date.isSame(start.value, 'day');
    };
    const isEnd = (dateObj) => {
      const date = dayjs(dateObj.date);
      return end.value && date.isSame(end.value, 'day');
    };

    const isSunday = (dateObj) => {
      const date = dayjs(dateObj.date);
      return date.day() === 0; 
    };

    const nextMonth = () => {
      currentMonth.value = currentMonth.value.add(1, 'month');
    };

    const prevMonth = () => {
      currentMonth.value = currentMonth.value.subtract(1, 'month');
    };

    const saveSelection = () => {
      emit('update-range', {
        startDate: start.value ? start.value.format('YYYY-MM-DD') : null,
        endDate: end.value ? end.value.format('YYYY-MM-DD') : null,
      });
    };

    return {
      currentMonthYear,
      daysOfWeek,
      dates,
      selectDate,
      isSelected,
      isRange,
      isStart,
      isEnd,
      nextMonth,
      prevMonth,
      saveSelection,
      isSunday,
    };
  },
};
</script>


<style scoped>
* {
  font-family: 'Raleway', sans-serif;
  font-weight: 600;
}
.month-year{
  text-transform: capitalize;
}
.week-margin{
  padding-bottom: 15px;
  padding-top: 10px;
}
.btn-nav-month, .point-day{
  cursor: pointer;
}
.point-day{
  border-radius: 4px;
  transition: 0.2s ease;
}
.btn-gotovo {
  width: 100%;
  padding: 11px 24px;
  background: rgb(17, 49, 71);
  color: white;
  font-weight: bold;
  text-align: center;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: 0.3s ease;
  margin-top: 14px;
}

.btn-gotovo:hover {
  background: rgb(17, 49, 71, 0.8); 
}
</style>