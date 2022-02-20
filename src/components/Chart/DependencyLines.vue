<!--
/**
 * @fileoverview DependencyLines component
 * @license MIT
 * @author Rafal Pospiech <neuronet.io@gmail.com>
 * @package GanttElastic
 */
-->
<template>
  <svg
    x="0"
    y="0"
    width="100%"
    height="100%"
    class="gantt-elastic__chart-dependency-lines-container"
    :style="{ ...root.style['chart-dependency-lines-container'] }"
  >
    <defs>
      <marker id="gantt-elastic__chart-dependency-lines-marker"
            :style="{...root.style['chart-dependency-lines-marker']}"
            viewBox="0 0 10 10"
            refX="10" refY="5"
            markerUnits="strokeWidth"
            markerWidth="4" markerHeight="4"
            orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z"></path>
      </marker>
    </defs>
    <g v-for="task in dependencyTasks" :key="task.id" :task="task">
      <path
        class="gantt-elastic__chart-dependency-lines-path"
        :style="{ ...root.style['chart-dependency-lines-path'], ...task.style['chart-dependency-lines-path'], ...task.style['chart-dependency-lines-path-' + dependencyLine.prevTaskId] }"
        v-for="dependencyLine in task.dependencyLines"
        :key="dependencyLine.id"
        :task="task"
        :d="dependencyLine.points"
        @click.stop="onDependencyLineClick($event, dependencyLine)"
        marker-end="url(#gantt-elastic__chart-dependency-lines-marker)"
      ></path>
    </g>
  </svg>
</template>

<script>
export default {
  name: 'DependencyLines',
  inject: ['root'],
  props: ['tasks'],
  data() {
    return {};
  },
  methods: {
    /**
     * Get path points
     *
     * @param {any} fromTaskId
     * @param {any} toTaskId
     * @param {any} dependencyType
     * @returns {string|null}
     */
    getPoints(fromTaskId, toTaskId, dependencyType) {
      const fromTask = this.root.getTask(fromTaskId);
      const toTask = this.root.getTask(toTaskId);
      if (
        fromTask === null ||
        toTask === null ||
        !this.root.isTaskVisible(toTask) ||
        !this.root.isTaskVisible(fromTask)
      ) {
        return null;
      }

      let startX, stopX;
      switch (dependencyType) {
        case 'startToStart':
          startX = fromTask.xP;
          stopX = toTask.xP;
          break;
        case 'endToEnd':
          startX = fromTask.xP + fromTask.widthP;
          stopX = toTask.xP + toTask.widthP;
          break;
        case 'startToEnd':
          startX = fromTask.xP;
          stopX = toTask.xP + toTask.widthP;
          break;
        case 'endToStart':
        default:
          startX = fromTask.xP + fromTask.widthP;
          stopX = toTask.xP;
          break;
      }
      const startY = fromTask.yP + fromTask.height / 2;
      const stopY = toTask.yP + toTask.height / 2;

      const distanceX = stopX - startX;
      let distanceY;
      let yMultiplier = 1;
      if (stopY >= startY) {
        distanceY = stopY - startY;
      } else {
        distanceY = startY - stopY;
        yMultiplier = -1;
      }
      const offset = 10;
      const roundness = 4;
      const isBefore = distanceX <= offset + roundness;
      let points = `M ${startX} ${startY}
          L ${startX + offset},${startY} `;
      if (isBefore) {
        points += `Q ${startX + offset + roundness},${startY} ${startX + offset + roundness},${startY +
          roundness * yMultiplier}
            L ${startX + offset + roundness},${startY + (distanceY * yMultiplier) / 2 - roundness * yMultiplier}
            Q ${startX + offset + roundness},${startY + (distanceY * yMultiplier) / 2} ${startX + offset},${startY +
          (distanceY * yMultiplier) / 2}
            L ${startX - offset + distanceX},${startY + (distanceY * yMultiplier) / 2}
            Q ${startX - offset + distanceX - roundness},${startY + (distanceY * yMultiplier) / 2} ${startX -
          offset +
          distanceX -
          roundness},${startY + (distanceY * yMultiplier) / 2 + roundness * yMultiplier}
            L ${startX - offset + distanceX - roundness},${stopY - roundness * yMultiplier}
            Q ${startX - offset + distanceX - roundness},${stopY} ${startX - offset + distanceX},${stopY}
            L ${stopX},${stopY}`;
      } else {
        points += `L ${startX + distanceX / 2 - roundness},${startY}
            Q ${startX + distanceX / 2},${startY} ${startX + distanceX / 2},${startY + roundness * yMultiplier}
            L ${startX + distanceX / 2},${stopY - roundness * yMultiplier}
            Q ${startX + distanceX / 2},${stopY} ${startX + distanceX / 2 + roundness},${stopY}
            L ${stopX},${stopY}`;
      }
      return points;
    },
    onDependencyLineClick(event, dependencyLine){
      if (this.root.state.refs.dependencyPopupEl) {
        this.root.state.popupData = {
          taskId: dependencyLine.taskId,
          prevTaskId: dependencyLine.prevTaskId,
          dependencyLineEl: event.target
        };
      }
    }
  },
  computed: {
    /**
     * Get tasks which are dependent on other tasks
     *
     * @returns {array}
     */
    dependencyTasks() {
      return this.tasks
        .filter(task => typeof task.dependentOn !== 'undefined')
        .map(task => {
          task.dependencyLines = task.dependentOn.map(item => {
            return { 
              points: this.getPoints(item.previousTask, task.id, item.dependencyType),
              prevTaskId: item.previousTask,
              taskId: task.id,
            };
          });
          return task;
        })
        .filter(task => task.dependencyLines.points !== null);
    }
  }
};
</script>
