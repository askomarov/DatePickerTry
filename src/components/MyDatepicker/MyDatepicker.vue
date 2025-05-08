<script setup lang="ts">
import { ref, computed } from 'vue'
import { startOfWeek, format, addMonths, isBefore, addDays } from 'date-fns'
import { ru } from 'date-fns/locale'
import { onClickOutside } from '@vueuse/core'
import CalendarMonth from './CalendarMonth.vue'

interface Props {
  minDate?: Date
  showTwoMonths?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  minDate: () => new Date(),
  showTwoMonths: false,
})

// Используем defineModel вместо props+emit+watch
const dates = defineModel<[Date | null, Date | null]>('modelValue', { default: () => [null, null] })

const isOpen = ref(false)
const currentDate = ref(new Date())
const hoveredDate = ref<Date | null>(null)
const noReturnTicket = ref(false)
const datepickerRef = ref(null)

// Добавляем обработчик клика вне компонента
onClickOutside(datepickerRef, () => {
  if (isOpen.value) {
    isOpen.value = false
  }
})

// Computed-свойства для доступа к началу и концу диапазона дат
const selectedStartDate = computed({
  get: () => dates.value[0],
  set: (value) => {
    dates.value = [value, dates.value[1]]
  },
})

const selectedEndDate = computed({
  get: () => dates.value[1],
  set: (value) => {
    dates.value = [dates.value[0], value]
  },
})

// Получаем локализованные названия дней недели из date-fns
const weekDays = computed(() => {
  const weekStart = startOfWeek(new Date(), { locale: ru })
  return Array.from({ length: 7 }).map((_, i) => {
    const day = addDays(weekStart, i)
    return format(day, 'EEEEEE', { locale: ru }) // 'EEEEEE' - сокращенная форма названия дня недели (Пн, Вт, ...)
  })
})

const toggleCalendar = () => {
  isOpen.value = !isOpen.value
  noReturnTicket.value = false
}

const nextMonth = () => {
  currentDate.value = addMonths(currentDate.value, 1)
}

const prevMonth = () => {
  currentDate.value = addMonths(currentDate.value, -1)
}

const selectDate = (date: Date) => {
  if (isBefore(date, props.minDate)) return

  if (noReturnTicket.value) {
    dates.value = [date, null]
    isOpen.value = false
    return
  }

  if (!dates.value[0] || dates.value[1]) {
    // Если начальная дата не выбрана или уже выбраны обе даты,
    // выбираем только начальную дату
    dates.value = [date, null]
  } else {
    // Иначе выбираем конечную дату
    if (date < dates.value[0]) {
      // Если выбранная дата раньше начальной, меняем их местами
      dates.value = [date, dates.value[0]]
    } else {
      dates.value = [dates.value[0], date]
    }
  }
}

const formatSelectedDates = computed(() => {
  if (!selectedStartDate.value) return 'Выберите даты'

  const start = format(selectedStartDate.value, 'd MMM', { locale: ru })
  if (!selectedEndDate.value) return start

  const end = format(selectedEndDate.value, 'd MMM', { locale: ru })
  return `${start} - ${end}`
})

const handleNoReturnTicket = () => {
  if (!dates.value[0]) return

  noReturnTicket.value = true
  dates.value = [dates.value[0], null]
  isOpen.value = false
}

const handleSelectDate = (date: Date) => {
  selectDate(date)
}

const handleHoverDate = (date: Date | null) => {
  hoveredDate.value = date
}
</script>

<template>
  <div class="datepicker" ref="datepickerRef">
    <div class="datepicker__input" @click="toggleCalendar">
      {{ formatSelectedDates }}
    </div>

    <div
      v-if="isOpen"
      class="datepicker__calendar"
      :class="{ 'datepicker__calendar--two-months': showTwoMonths }"
    >
      <div class="datepicker__header">
        <button class="datepicker__nav-btn" @click="prevMonth">&lt;</button>
        <span class="datepicker__month">
          {{ format(currentDate, 'LLLL yyyy', { locale: ru }) }}
        </span>
        <span v-if="showTwoMonths" class="datepicker__month">
          {{ format(addMonths(currentDate, 1), 'LLLL yyyy', { locale: ru }) }}
        </span>
        <button class="datepicker__nav-btn" @click="nextMonth">&gt;</button>
      </div>
      <div class="datepicker__calendar-content">
        <CalendarMonth
          :base-date="currentDate"
          :hovered-date="hoveredDate"
          :selected-start-date="selectedStartDate"
          :selected-end-date="selectedEndDate"
          :min-date="props.minDate"
          :no-return-ticket="noReturnTicket"
          :week-days="weekDays"
          @select-date="handleSelectDate"
          @hover-date="handleHoverDate"
          class="datepicker__month-container"
        />

        <CalendarMonth
          v-if="showTwoMonths"
          :base-date="addMonths(currentDate, 1)"
          :hovered-date="hoveredDate"
          :selected-start-date="selectedStartDate"
          :selected-end-date="selectedEndDate"
          :min-date="props.minDate"
          :no-return-ticket="noReturnTicket"
          :week-days="weekDays"
          @select-date="handleSelectDate"
          @hover-date="handleHoverDate"
          class="datepicker__month-container"
        />
      </div>
      <button
        class="datepicker__no-return"
        :disabled="!selectedStartDate"
        @click="handleNoReturnTicket"
      >
        Вторая дата не нужна
      </button>
    </div>
  </div>
</template>

<style scoped>
.datepicker {
  position: relative;
  width: 300px;
}

.datepicker__input {
  padding: 12px 16px;
  background: white;
  border-radius: 8px;
  cursor: pointer;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
}

.datepicker__input:hover {
  background: #f7f9fc;
}

.datepicker__no-return {
  padding: 8px 16px;
  background: none;
  border: 1px solid #00b1df;
  border-radius: 4px;
  color: #00b1df;
  cursor: pointer;
  width: 100%;
}

.datepicker__no-return:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.datepicker__no-return:not(:disabled):hover {
  background: #f1f4f8;
}

.datepicker__calendar {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  width: 100%;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  padding: 16px;
  z-index: 100;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.datepicker__calendar--two-months {
  width: 616px;

  .datepicker__calendar-content {
    display: grid;
    gap: 12px;
    grid-template-columns: 1fr 1fr;
  }
}

.datepicker__month-container {
  width: 100%;
}

.datepicker__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.datepicker__month {
  font-weight: 600;
  text-transform: capitalize;
}

.datepicker__nav-btn {
  border: none;
  background: none;
  padding: 4px 8px;
  cursor: pointer;
  color: #0f1d40;
}

.datepicker__nav-btn:hover {
  background: #f1f4f8;
  border-radius: 4px;
}
</style>
