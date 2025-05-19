<script setup lang="ts">
import { ref, watch } from 'vue'
import DatePicker from './components/MyDatepicker/MyDatepicker.vue'
import MyDatepickerTwoInputs from './components/MyDatepicker/MyDatepickerTwoInputs.vue'
import InlineDatePicker from './components/InlineDatePicker/InlineDatePicker.vue'

const selectedDates = ref<[Date | null, Date | null]>([null, null])
const singleSelectedDate = ref<Date | null>(null)
const singleReturnDateWithCustomFooter = ref<Date | null>(null)
// Состояние календаря для демонстрации v-model:open
const isCalendarStartDateOpen = ref(false)
const isCalendarBackDateOpen = ref(false)
const isButtonCloseCalendarDisabled = ref(true)

// watch за singleSelectedDate  если не null но разблокируем кнопку
watch(singleSelectedDate, (newValue) => {
  if (newValue) {
    isButtonCloseCalendarDisabled.value = false
  } else {
    isButtonCloseCalendarDisabled.value = true
  }
})
// Этот метод вызывается из внешнего кода
const handleNoReturnTicket = () => {
  singleReturnDateWithCustomFooter.value = null
  isCalendarStartDateOpen.value = false
  isCalendarBackDateOpen.value = false
}
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
      <br />
      <hr />
      <br />

      <!-- Пример использования v-model:open -->
      <p class="text-lg font-semibold">InlineDatePicker</p>
      <p>c использованием v-model:open</p>
      <div class="selected-dates mb-2 font-semibold">
        <div v-if="!singleSelectedDate">Одиночная дата не выбрана (с кнопкой)</div>
        <div v-else>Выбранная дата: {{ singleSelectedDate.toLocaleDateString() }}</div>
      </div>
      <InlineDatePicker
        v-model="singleSelectedDate"
        v-model:open="isCalendarStartDateOpen"
        :min-date="new Date()"
        :show-two-months="true"
        :close-on-select="false"
      >
        <template #footer>
          <button
            class="no-return-btn"
            :disabled="isButtonCloseCalendarDisabled"
            @click="handleNoReturnTicket"
          >
            Обраный билет не нужен
          </button>
        </template>
      </InlineDatePicker>

      <br />
      <hr />
      <br />


      <div class="selected-dates mb-2 font-semibold">
        <div v-if="!singleReturnDateWithCustomFooter">Дата обратно</div>
        <div v-else>
          Выбранная дата обратно:
          {{ singleReturnDateWithCustomFooter.toLocaleDateString() }}
        </div>

        <div class="mt-2">
          Календарь сейчас:
          {{ isCalendarBackDateOpen ? 'открыт' : 'закрыт' }} (isCalendarBackDateOpen =
          {{ isCalendarBackDateOpen }})
        </div>
      </div>
      <InlineDatePicker
        v-model="singleReturnDateWithCustomFooter"
        v-model:open="isCalendarBackDateOpen"
        :min-date="new Date()"
        :show-two-months="true"
      >
        <template #footer>
          <button
            class="no-return-btn"
            :disabled="isButtonCloseCalendarDisabled"
            @click="handleNoReturnTicket"
          >
            Обраный билет не нужен
          </button>
        </template>
      </InlineDatePicker>
    </div>
  </main>
</template>

<style scoped>
.no-return-btn {
  padding: 8px 16px;
  background: none;
  border: 1px solid #00b1df;
  border-radius: 4px;
  color: #00b1df;
  cursor: pointer;
  width: 100%;

  &:disabled {
    background: #f1f4f8;
    color: #ccc;
    cursor: not-allowed;
  }
}

.no-return-btn:hover {
  background: #f1f4f8;
}

.external-control-btn {
  margin-top: 8px;
  padding: 8px 16px;
  background: #00b1df;
  border-radius: 4px;
  color: white;
  cursor: pointer;
  border: none;
}

.external-control-btn:hover {
  background: #009ac2;
}
</style>
