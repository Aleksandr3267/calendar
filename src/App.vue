<template>
  <div class="flex justify-center items-center h-screen bg-gray-100">
    <Calendar @update-range="handleUpdateRange" :startDate="startDate" :endDate="endDate" />
  </div>
</template>

<script>
import Calendar from "./components/Calendar.vue";
export default {
  components: {
    Calendar,
  },
  data() {
    return {
      startDate: null,
      endDate: null,
    };
  },
  methods: {
    handleUpdateRange(range) {
      this.startDate = range.startDate;
      this.endDate = range.endDate;
      const finishDate = `Начальная дата: ${this.startDate} Конечная дата: ${this.endDate ? this.endDate : 'не выбрана'}`;
      console.log('Выбранный диапазон:', finishDate);
    },
  },
};
</script>
