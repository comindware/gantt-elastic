<!--
/**
 * @fileoverview Chart component
 * @license MIT
 * @author Rafal Pospiech <neuronet.io@gmail.com>
 * @package GanttElastic
 */
-->
<template>
  <div class="gantt-elastic__chart" :style="{ ...root.style['chart'] }" ref="chart">
    <div
      class="gantt-elastic__chart-calendar-container"
      ref="chartCalendarContainer"
      :style="{
        ...root.style['chart-calendar-container'],
        height: root.state.options.calendar.height + 'px',
        'margin-bottom': root.state.options.calendar.gap + 'px'
      }"
    >
      <calendar 
        :calendar="root.state.options.calendar" 
        :times="root.state.options.times" 
        :width="root.state.options.width" 
        :locale="root.state.options.locale"
      >
      </calendar>
    </div>
    <div
      class="gantt-elastic__chart-graph-container"
      ref="chartGraphContainer"
      :style="{
        ...root.style['chart-graph-container'],
        height: root.state.options.height - root.state.options.calendar.height + 'px'
      }"
    >
      <div
        :style="{
          ...root.style['chart-area'],
          width: this.root.state.options.width + 'px',
          'min-width': '100%',
          height: root.state.options.rowsHeight + 'px'
        }"
      >
        <div
          class="gantt-elastic__chart-graph"
          ref="chartGraph"
          :style="{ ...root.style['chart-graph'], height: '100%', 'min-width': '100%'}"
        >
          <svg
            class="gantt-elastic__chart-graph-svg"
            :style="{ ...root.style['chart-graph-svg'], 'min-width': '100%'}"
            ref="chartGraphSvg"
            x="0"
            y="0"
            :height="root.state.options.allVisibleTasksHeight + 'px'"
            xmlns="http://www.w3.org/2000/svg"
          >
            <days-highlight></days-highlight>
            <grid></grid>
            <g
              class="gantt-elastic__chart-row-wrapper"
              :style="{ ...root.style['chart-row-wrapper'] }"
              v-for="task in root.visibleTasks"
              :task="task"
              :key="task.id"
            >
              <task-planned :task="task" v-if="task.durationPlanned > 0"></task-planned>
              <component :task="task" v-if="task.duration > 0 && root.state.options.showExpected" :is="task.type"></component>
            </g>
            <dependency-lines :tasks="root.visibleTasks"></dependency-lines>
          </svg>
          <dependency-popup></dependency-popup>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Grid from './Grid.vue';
import DaysHighlight from './DaysHighlight.vue';
import Calendar from '../Calendar/Calendar.vue';
import DependencyLines from './DependencyLines.vue';
import DependencyPopup from './DependencyPopup.vue';
import Task from './Row/Task.vue';
import TaskPlanned from './Row/TaskPlanned.vue';
import Milestone from './Row/Milestone.vue';
import Project from './Row/Project.vue';
export default {
  name: 'Chart',
  components: {
    Grid,
    DependencyLines,
    DependencyPopup,
    Calendar,
    Task,
    TaskPlanned,
    Milestone,
    Project,
    DaysHighlight,
  },
  inject: ['root'],
  data() {
    return {
      moving: false,
    };
  },
  /**
   * Mounted
   */
  mounted() {
    this.root.state.refs.chart = this.$refs.chart;
    this.root.state.refs.chartCalendarContainer = this.$refs.chartCalendarContainer;
    this.root.state.refs.chartGraphContainer = this.$refs.chartGraphContainer;
    this.root.state.refs.chartGraph = this.$refs.chartGraph;
    this.root.state.refs.chartGraphSvg = this.$refs.chartGraphSvg;
  },

  computed: {
    /**
     * Get view box
     *
     * @returns {string}
     */
    getViewBox() {
      return `0 0 ${this.root.state.options.width} ${this.root.state.options.allVisibleTasksHeight}`;
    }
  }
};
</script>
