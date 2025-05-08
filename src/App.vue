<script setup lang="ts">
import { ref } from 'vue'
import DatePicker from './components/MyDatepicker/MyDatepicker.vue'
import MyDatepickerTwoInputs from './components/MyDatepicker/MyDatepickerTwoInputs.vue'

const selectedDates = ref<[Date | null, Date | null]>([null, null])
</script>

<template>
  <main>
    <div class="container">
      <div class="selected-dates mb-2 font-semibold">
        <div v-if="!selectedDates[0]">Даты не выбраны</div>
        <div v-else>
          Выбранные даты: {{ selectedDates[0]?.toLocaleDateString() }}
          <template v-if="selectedDates[1]">
            - {{ selectedDates[1]?.toLocaleDateString() }}
          </template>
        </div>
      </div>
      <MyDatepickerTwoInputs
        v-model="selectedDates"
        :min-date="new Date()"
        :show-two-months="true"
      />
      <br />
      <hr />
      <br />
      <DatePicker v-model="selectedDates" :min-date="new Date()" :show-two-months="true" />
      <br />
      <hr />
      <br />
      <DatePicker v-model="selectedDates" :min-date="new Date()" :show-two-months="false" />
    </div>
  </main>
</template>

<style scoped></style>
