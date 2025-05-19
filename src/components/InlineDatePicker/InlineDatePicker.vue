<script setup lang="ts">
import { ref, computed } from 'vue'
import { startOfWeek, format, addMonths, isBefore, addDays } from 'date-fns'
import { ru } from 'date-fns/locale'
import { onClickOutside } from '@vueuse/core'
import InlineCalendarMonth from './InlineCalendarMonth.vue'

interface Props {
  modelValue?: Date | null // Для v-model выбранной даты
  open?: boolean // Для v-model:open
  minDate?: Date
  showTwoMonths?: boolean
  closeOnSelect?: boolean // Закрывать календарь после выбора даты
}

const props = withDefaults(defineProps<Props>(), {
  minDate: () => new Date(),
  showTwoMonths: false,
  closeOnSelect: true,
})

// Модель для выбранной даты
const selectedDate = defineModel<Date | null>('modelValue', { default: null })

// Модель для состояния открытия
const isOpen = defineModel<boolean>('open', { default: false })

const currentDate = ref(new Date())
const hoveredDate = ref<Date | null>(null)
const datepickerRef = ref(null)

// Обработчик клика вне компонента
onClickOutside(datepickerRef, () => {
  if (isOpen.value) {
    isOpen.value = false
  }
})

// Получаем локализованные названия дней недели
const weekDays = computed(() => {
  const weekStart = startOfWeek(new Date(), { locale: ru })
  return Array.from({ length: 7 }).map((_, i) => {
    const day = addDays(weekStart, i)
    return format(day, 'EEEEEE', { locale: ru })
  })
})

const toggleCalendar = () => {
  isOpen.value = !isOpen.value
}

const nextMonth = () => {
  currentDate.value = addMonths(currentDate.value, 1)
}

const prevMonth = () => {
  currentDate.value = addMonths(currentDate.value, -1)
}

const selectDate = (date: Date) => {
  if (isBefore(date, props.minDate)) return
  selectedDate.value = date
  if(props.closeOnSelect) {
    isOpen.value = false // Закрываем календарь после выбора даты
  }
}

const formatSelectedDate = computed(() => {
  if (!selectedDate.value) return 'Выберите дату'
  return format(selectedDate.value, 'd MMMM yyyy', { locale: ru })
})

const handleSelectDate = (date: Date) => {
  selectDate(date)
}

const handleHoverDate = (date: Date | null) => {
  hoveredDate.value = date
}
</script>

<template>
  <div class="inline-datepicker" ref="datepickerRef">
    <div class="inline-datepicker__input" @click="toggleCalendar">
      <!-- Удалим отладочную информацию из интерфейса -->
      {{ formatSelectedDate }}
    </div>

    <div
      v-if="isOpen"
      class="inline-datepicker__calendar"
      :class="{ 'inline-datepicker__calendar--two-months': showTwoMonths }"
    >
      <div class="inline-datepicker__header">
        <span class="inline-datepicker__month">
          {{ format(currentDate, 'yyyy', { locale: ru }) }}
        </span>
        <div class="inline-datepicker__nav">
          <button class="inline-datepicker__nav-btn" @click="prevMonth">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
            >
              <path
                d="M14.7 17.3C14.3134 17.6866 13.6866 17.6866 13.3 17.3L8.70711 12.7071C8.31658 12.3166 8.31658 11.6834 8.70711 11.2929L13.3 6.7C13.6866 6.3134 14.3134 6.3134 14.7 6.7C15.0866 7.0866 15.0866 7.7134 14.7 8.1L10.8 12L14.7 15.9C15.0866 16.2866 15.0866 16.9134 14.7 17.3Z"
                fill="currentColor"
              />
            </svg>
          </button>
          <span class="inline-datepicker__month">
            {{ format(currentDate, 'LLLL', { locale: ru }) }}
          </span>
          <!-- <span v-if="showTwoMonths" class="inline-datepicker__month">
            {{ format(addMonths(currentDate, 1), 'LLLL yyyy', { locale: ru }) }}
          </span> -->
          <button class="inline-datepicker__nav-btn" @click="nextMonth">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
            >
              <path
                d="M12.6 12L8.7 8.1C8.3134 7.7134 8.3134 7.0866 8.7 6.7C9.0866 6.3134 9.7134 6.3134 10.1 6.7L14.6929 11.2929C15.0834 11.6834 15.0834 12.3166 14.6929 12.7071L10.1 17.3C9.7134 17.6866 9.0866 17.6866 8.7 17.3C8.3134 16.9134 8.3134 16.2866 8.7 15.9L12.6 12Z"
                fill="currentColor"
              />
            </svg>
          </button>
        </div>
      </div>
      <div class="inline-datepicker__calendar-content">
        <InlineCalendarMonth
          :base-date="currentDate"
          :hovered-date="hoveredDate"
          :selected-date="selectedDate"
          :min-date="props.minDate"
          :week-days="weekDays"
          @select-date="handleSelectDate"
          @hover-date="handleHoverDate"
          class="inline-datepicker__month-container"
        />

        <InlineCalendarMonth
          v-if="showTwoMonths"
          :base-date="addMonths(currentDate, 1)"
          :hovered-date="hoveredDate"
          :selected-date="selectedDate"
          :min-date="props.minDate"
          :week-days="weekDays"
          @select-date="handleSelectDate"
          @hover-date="handleHoverDate"
          class="inline-datepicker__month-container inline-datepicker__month-container--disabled"
        />
      </div>

      <!-- Добавляем слот под списком дней -->
      <div class="inline-datepicker__footer">
        <!-- <slot name="footer" :handle-no-return-ticket="handleNoReturnTicket"></slot> -->
        <slot name="footer"></slot>
      </div>
    </div>
  </div>
</template>

<style scoped>
.inline-datepicker {
  position: relative;
  width: 300px;
}

.inline-datepicker__input {
  padding: 12px 16px;
  background: white;
  border-radius: 8px;
  margin-bottom: 8px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.inline-datepicker__input:hover {
  background: #f7f9fc;
}

.inline-datepicker__calendar {
  width: 100%;
  background: #f4f5f8;
  border-radius: 16px;

  padding: 12px;
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.inline-datepicker__calendar--two-months {
  /* width: 616px; */
  overflow-x: clip;

  .inline-datepicker__calendar-content {
    display: grid;
    gap: 12px;
    grid-template-columns: 1fr 1fr;
  }
}

.inline-datepicker__month-container {
  background-color: #fff;
  padding: 8px;
  width: 100%;
  border-radius: 12px;
  width: 192px;
  font-size: 14px;
  font-weight: 500;
  line-height: 1;
}

.inline-datepicker__month-container--disabled {
  opacity: 0.2;
  color: #515151;
  pointer-events: none;
  user-select: none;
}

.inline-datepicker__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-inline: 8px;
}

.inline-datepicker__month {
  font-weight: 600;
  text-transform: capitalize;
}

.inline-datepicker__nav{
  display: flex;
  align-items: center;
  gap: 2px;
}

.inline-datepicker__nav-btn {
  border: none;
  background: none;
  padding: 4px 8px;
  cursor: pointer;
  color: #0f1d40;
}

.inline-datepicker__nav-btn:hover {
  background: #f1f4f8;
  border-radius: 4px;
}

.inline-datepicker__footer {
  margin-top: 12px;
}
</style>
