<template>
  <div class="panel">
    <div class="panel__header common-border-rb">星期/时间</div>
    <div class="panel__time-range common-border-rb">00:00 - 12:00</div>
    <div class="panel__time-range common-border-rb">12:00 - 24:00</div>
    <div class="panel__time-list">
      <div v-for="hour in hours" class="panel__time-item common-border-rb" :key="hour">
        {{ hour }}
      </div>
    </div>
    <ol class="panel__day-list">
      <li class="panel__day-name common-border-rb" v-for="(day, index) in dayName" :key="index">
        {{ day }}
      </li>
    </ol>

    <div
      class="panel__schedule-list"
      @mousedown="onMousedownStartSelect"
      @mouseup="onMouseupEndSelect"
      @mousemove="onMousemoveUpdateSelect"
      @mouseleave="onMouseleaveEndSelect"
    >
      <template v-for="(scheduleArr, dayIndex) in scheduleStatus">
        <li
          v-for="(schedule, scheduleIndex) in scheduleArr"
          :data-day="dayIndex"
          :data-schedule="scheduleIndex"
          class="panel__schedule-item common-border-rb"
          :class="{ 'panel__schedule-item--selected': schedule }"
          :key="`${dayIndex}-${scheduleIndex}`"
        ></li>
      </template>

      <div v-if="isDragSelect" class="drag-block" :style="dragBlockStyle"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { DAY_NAME } from '@/const/scheduleConfig'
import { isDef } from '@/util'
import { throttle, clone } from 'lodash-es'

const props = defineProps({
  scheduleStatus: Array,
})
const emits = defineEmits(['update:scheduleStatus'])

const hours = Array.from({ length: 24 }, (_, index) => index)
const dayName = DAY_NAME

const onClickSelectSchedule = (day, schedule) => {
  const scheduleStatusCp = clone(props.scheduleStatus)

  scheduleStatusCp[day][schedule] = true

  emits('update:scheduleStatus', scheduleStatusCp)
}

// drag select schedule block
let isDragSelect = ref(false)
let dragBlockStyle = computed(() => {
  const { startDay, startSchedule, endDay = null, endSchedule = null } = dragData
  const dayStart = Math.min(startDay, endDay)
  const dayLength = endDay === null ? 0 : Math.abs(startDay - endDay) + 1
  const scheduleStart = Math.min(startSchedule, endSchedule)
  const scheduleLength = endSchedule === null ? 0 : Math.abs(startSchedule - endSchedule) + 1

  return {
    position: 'absolute',
    background: '#4184f2',
    opacity: '0.7',
    top: `${dayStart * 33}px`,
    left: `${scheduleStart * 13}px`,
    width: `${scheduleLength * 13}px`,
    height: `${dayLength * 33}px`,
  }
})

const dragData = reactive({
  startDay: null,
  startSchedule: null,
  endDay: null,
  endSchedule: null,
})

/**
 * Get start day and schedule when mousedown
 * @param $event
 */
const onMousedownStartSelect = ($event) => {
  const { day, schedule } = $event.target.dataset

  dragData.startDay = day
  dragData.startSchedule = schedule
  isDragSelect.value = true
}

/**
 * Update current day and schedule by mousemove event
 * @param $event
 */
const onMousemoveUpdateSelect = ($event) => {
  if (!isDragSelect.value) return

  const { day, schedule } = $event.target.dataset

  throttleUpdateDragEndData(day, schedule)
}

/**
 * End drag select, update scheduleStatus and reset dragData
 * @type {function(): (*)}
 */
const throttleUpdateDragEndData = throttle((day, schedule) => {
  if (isDef(day) && isDef(schedule)) {
    dragData.endDay = day
    dragData.endSchedule = schedule
  }
}, 50)

const onMouseupEndSelect = ($event) => {
  isDragSelect.value = false
  const { day, schedule } = $event.target.dataset

  // If start cell same with end cell, handle it like a click event
  if (day === dragData.startDay && schedule === dragData.startSchedule) {
    onClickSelectSchedule(day, schedule)
  } else {
    updateDragSelectSchedule()
  }

  resetDragData()
}

/**
 * End the select with current status, when mouse leave select area
 */
const onMouseleaveEndSelect = () => {
  isDragSelect.value = false
  updateDragSelectSchedule()
  resetDragData()
}

const updateDragSelectSchedule = () => {
  const { startDay, startSchedule, endDay, endSchedule } = dragData
  const dayStart = Math.min(startDay, endDay)
  const dayLength = endDay === null ? 0 : Math.abs(startDay - endDay) + 1
  const scheduleStart = Math.min(startSchedule, endSchedule)
  const scheduleLength = endSchedule === null ? 0 : Math.abs(startSchedule - endSchedule) + 1
  const scheduleStatusCp = clone(props.scheduleStatus)

  for (let i = dayStart; i < dayStart + dayLength; i++) {
    for (let j = scheduleStart; j < scheduleStart + scheduleLength; j++) {
      scheduleStatusCp[i][j] = true
    }
  }

  emits('update:scheduleStatus', scheduleStatusCp)
}

const resetDragData = () => {
  for (const key in dragData) {
    dragData[key] = null
  }
}
</script>

<style lang="scss" scoped>
.panel {
  display: grid;
  grid-template-rows: 48px 32px 231px;
  grid-template-columns: 160px repeat(2, 312px);

  &__header {
    font-weight: bold;
    grid-row-start: 1;
    grid-row-end: 3;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  &__time-range {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  &__time-list {
    grid-column-start: 2;
    grid-column-end: 4;
    display: flex;
  }

  &__time-item {
    width: 25px;
    line-height: 32px;
  }

  &__day-list,
  &__schedule-item {
    list-style-type: none;
  }

  &__day-name {
    line-height: 32px;
  }

  &__schedule-list {
    display: flex;
    flex-wrap: wrap;
    grid-column-start: 2;
    grid-column-end: 4;
    position: relative;
    overflow: hidden;
  }

  &__schedule-item {
    flex: 0 0 12px;
    height: 32px;

    &:not(.panel__schedule-item--selected):hover {
      background: #eee;
    }

    &--selected {
      background: $themeColor;
    }
  }
}
</style>
