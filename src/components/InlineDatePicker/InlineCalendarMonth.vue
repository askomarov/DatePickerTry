<script setup lang="ts">
import { computed } from 'vue'
import {
  startOfMonth,
  endOfMonth,
  startOfWeek,
  endOfWeek,
  eachDayOfInterval,
  format,
  isSameDay,
  isBefore,
} from 'date-fns'
import { ru } from 'date-fns/locale'

interface Props {
  baseDate: Date
  hoveredDate: Date | null
  selectedDate: Date | null
  minDate: Date
  weekDays: string[]
  disabled?: boolean
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: 'selectDate', date: Date): void
  (e: 'hoverDate', date: Date | null): void
}>()

const getCalendarDays = (date: Date) => {
  const start = startOfWeek(startOfMonth(date), { locale: ru })
  const end = endOfWeek(endOfMonth(date), { locale: ru })
  return eachDayOfInterval({ start, end })
}

const calendarDays = computed(() => getCalendarDays(props.baseDate))

const isSelected = (date: Date): boolean => {
  if (!props.selectedDate) return false
  return isSameDay(date, props.selectedDate)
}

const isDisabled = (date: Date): boolean => {
  return isBefore(date, props.minDate)
}

const isCurrentMonth = (date: Date): boolean => {
  if (!date || !props.baseDate) return false
  return date.getMonth() === props.baseDate.getMonth()
}

const handleDateClick = (date: Date) => {
  emit('selectDate', date)
}

const handleMouseEnter = (date: Date) => {
  emit('hoverDate', date)
}

const handleMouseLeave = () => {
  emit('hoverDate', null)
}
</script>

<template>
  <div class="calendar-month">
    <div class="calendar-month__weekdays">
      <div v-for="weekday in weekDays" :key="weekday" class="calendar-month__weekday">
        {{ weekday }}
      </div>
    </div>

    <div class="calendar-month__days">
      <div
        v-for="date in calendarDays"
        :key="date.toISOString()"
        class="calendar-month__day"
        :class="{
          'calendar-month__day--selected': isSelected(date),
          'calendar-month__day--disabled': isDisabled(date),
          'calendar-month__day--other-month': !isCurrentMonth(date),
        }"
        @click="handleDateClick(date)"
        @mouseenter="handleMouseEnter(date)"
        @mouseleave="handleMouseLeave"
      >
        {{ isCurrentMonth(date) ? format(date, 'd') : '' }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar-month__weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  margin-bottom: 8px;
  padding-inline: 4px;
}

.calendar-month__weekday {
  font-size: 12px;
  color: #78839c;
  text-align: center;
}

.calendar-month__days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  /* gap: 2px; */
  row-gap: 4px;
  padding-inline: 4px;
}

.calendar-month__day {
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-radius: 4px;
}

.calendar-month__day:hover:not(.calendar-month__day--disabled, .calendar-month__day--selected) {
  background: #f1f4f8;
}

.calendar-month__day--selected {
  background: #00b1df;
  color: white;
}

.calendar-month__day--selected:hover {
  background: #00a0cc;
}

.calendar-month__day--disabled {
  color: #ccc;
  cursor: not-allowed;
}

.calendar-month__day--disabled:hover {
  background: none;
}

.calendar-month__day--other-month {
  visibility: hidden;
  pointer-events: none;
}
</style>
