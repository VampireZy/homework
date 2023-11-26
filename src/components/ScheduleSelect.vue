<template>
  <div class="schedule-select-main">
    <div class="legend-panel common-border-rb">
      <div class="legend-panel__item" v-for="(legend, index) in legendStatus" :key="index">
        <span class="legend-panel__icon" :style="{ background: legend.color }"></span>
        <span class="legend-panel__desc">{{ legend.desc }}</span>
      </div>
    </div>

    <ScheduleSelectPanel v-model:scheduleStatus="scheduleStatus" />
    <ScheduleResult @clearSelectSchedule="clearSelectSchedule" :scheduleStatus="scheduleStatus" />
  </div>

  <div class="button-group">
    <button class="button-group__item" @click="onClickSelectWorkdaySchedule">工作日黄金时间</button>
    <button class="button-group__item" @click="onClickSelectWeekendSchedule">休息日黄金时间</button>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { LEGEND_STATUS } from '@/const/scheduleConfig'
import ScheduleSelectPanel from './ScheduleSelectPanel.vue'
import ScheduleResult from './ScheduleResult.vue'

const legendStatus = ref(LEGEND_STATUS)
const scheduleStatus = reactive(Array.from({ length: 7 }, () => Array.from({ length: 48 }, () => false)))

const clearSelectSchedule = () => {
  for (const day of scheduleStatus) {
    for (const time in day) {
      day[time] = false
    }
  }
}

const onClickSelectWorkdaySchedule = () => {
  clearSelectSchedule()

  for (const day in scheduleStatus) {
    for (const time in scheduleStatus[day]) {
      if (day < 5 && time >= 18 && time < 42) {
        scheduleStatus[day][time] = true
      }
    }
  }
}

const onClickSelectWeekendSchedule = () => {
  clearSelectSchedule()

  for (const day in scheduleStatus) {
    for (const time in scheduleStatus[day]) {
      if (day >= 5 && time >= 18 && time < 42) {
        scheduleStatus[day][time] = true
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.schedule-select-main {
  border: solid 1px $lineColor;
  border-right: none;
  margin-top: 48px;

  .legend-panel {
    display: flex;
    justify-content: flex-end;
    padding: 12px;

    &__item {
      display: inline-flex;
      align-items: center;
      margin-left: 12px;
    }

    &__icon {
      display: inline-block;
      width: 12px;
      height: 4px;
      border-radius: 12px;
      margin-right: 12px;
      border: solid 1px $lineColor;
    }
  }
}

.button-group {
  margin-top: 24px;
}
</style>
