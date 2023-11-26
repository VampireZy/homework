<template>
  <div class="result">
    <div class="result__header">
      <h3 class="result__title">{{ headerMsg }}</h3>
      <button v-if="isSelectSchedule" class="result__clear-btn" @click="onClickClearSelectSchedule">清空</button>
    </div>

    <div class="result__panel">
      <div v-for="(schedules, day) in selectScheduleDay" class="result__item" :key="day">
        <span class="result__day-name">{{ dayName[day] }}</span>
        <span class="result__schedule">
          {{ calcSchedulesStr(schedules) }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { some, forEach, isEmpty, indexOf } from 'lodash-es'
import { DAY_NAME } from '@/const/scheduleConfig'

const props = defineProps({
  scheduleStatus: Array,
})

const emits = defineEmits(['clearSelectSchedule'])
const dayName = DAY_NAME

const selectScheduleDay = computed(() => {
  const result = {}

  forEach(props.scheduleStatus, (day, index) => {
    if (some(day, (schedule) => schedule)) {
      result[index] = day
    }
  })

  return result
})
const isSelectSchedule = computed(() => !isEmpty(selectScheduleDay.value))
const headerMsg = computed(() => (isSelectSchedule.value ? '已选择时间段' : '可拖动鼠标选择时间段'))

const calcSchedulesStr = (schedules) => {
  let start = 0
  let end = 0
  let result = []

  while (start < 47 && end < 47) {
    start = indexOf(schedules, true, end)

    if (start === -1) break

    end = indexOf(schedules, false, start)

    if (end === -1) {
      end = 48
    }

    const startHour = Math.floor(start * 0.5)
    const startHourStr = startHour < 10 ? `0${startHour}` : `${startHour}`
    const endHour = Math.floor(end * 0.5)
    const endHourStr = endHour < 10 ? `0${endHour}` : `${endHour}`

    result.push(`${startHourStr}:${start % 2 === 0 ? '00' : '30'}~${endHourStr}:${end % 2 === 0 ? '00' : '30'}`)
  }

  return result.join('、 ')
}

const onClickClearSelectSchedule = () => {
  emits('clearSelectSchedule')
}
</script>

<style lang="scss" scoped>
.result {
  width: 783px;
  border-right: solid 1px $lineColor;

  &__header {
    position: relative;
  }

  &__title {
    padding: 12px 0;
  }

  &__clear-btn {
    position: absolute;
    right: 0;
    top: 6px;
    background: transparent;
    color: $themeColor;
    border: none;
  }

  &__panel {
    padding: 0 24px 12px 24px;
  }

  &__item {
    display: flex;
    margin-top: 12px;
  }

  &__day-name {
    flex: 0 0 60px;
    margin-right: 24px;
  }

  &__schedule {
    text-align: left;
  }
}
</style>
