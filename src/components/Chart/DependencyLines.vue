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
const dependencyTypes = {
  startToStart: 'startToStart',
  endToEnd: 'endToEnd',
  startToEnd: 'startToEnd',
  endToStart: 'endToStart'
}
const axis = {
  x: 'x',
  y: 'y'
}
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
        case dependencyTypes.startToStart:
          startX = fromTask.xP;
          stopX = toTask.xP;
          break;
        case dependencyTypes.endToEnd:
          startX = fromTask.xP + fromTask.widthP;
          stopX = toTask.xP + toTask.widthP;
          break;
        case dependencyTypes.startToEnd:
          startX = fromTask.xP;
          stopX = toTask.xP + toTask.widthP;
          break;
        case dependencyTypes.endToStart:
        default:
          startX = fromTask.xP + fromTask.widthP;
          stopX = toTask.xP;
          break;
      }
      const lineMargin = 4
      const startY = (fromTask.yP + fromTask.height / 2) + lineMargin;
      const stopY = (toTask.yP + toTask.height / 2) - lineMargin;

      let distanceX = stopX - startX;
      let distanceY;
      let yMultiplier = 1;
      if (stopY >= startY) {
        distanceY = stopY - startY;
      } else {
        distanceY = startY - stopY;
        yMultiplier = -1;
      }
      let xMultiplier = 1;
      if (stopX >= startX) {
        xMultiplier = -1
      }
      let offset = 10;
      let roundness = 4;
      const isBefore = distanceX <= offset + roundness;
      let startMove = 0;
      if (!isBefore && (dependencyType === dependencyTypes.endToEnd)) {
        offset += distanceX;
      } else if (isBefore && dependencyType === dependencyTypes.startToStart) {
        offset -= distanceX; 
      }
      if (dependencyType === dependencyTypes.startToStart || dependencyType === dependencyTypes.startToEnd) {
        startMove -= offset;
      } else {
        startMove += offset
      }
      const cursour = {x: 0, y: 0};
      let points = `
        ${this.moveCursor(cursour, startX, startY)} 
        ${this.getLine(cursour, startMove, 0)}`;
      if (dependencyType === dependencyTypes.startToStart) {
        xMultiplier = -1
      } else if (dependencyType === dependencyTypes.endToEnd || dependencyType === dependencyTypes.endToStart) {
        xMultiplier = 1
      }
      points += `
        ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.y)}`
      distanceX = stopX - cursour.x;
      distanceY -= roundness;
      if (distanceX >= 0) {
        xMultiplier = 1
      } else {
        xMultiplier = -1
      }
      if (isBefore && dependencyType === dependencyTypes.endToStart) {
        points += `
          ${this.getLine(cursour, 0, (((distanceY - toTask.height / 2)) - roundness) * yMultiplier)}
          ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.x)}
          ${this.getLine(cursour, distanceX - 10 , 0)}
          ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.y)}`;
          xMultiplier = xMultiplier * -1;
      } else if (!isBefore && dependencyType === dependencyTypes.startToEnd) {
        points += `
          ${this.getLine(cursour, 0, (((distanceY - toTask.height / 2)) - roundness) * yMultiplier)}
          ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.x)}
          ${this.getLine(cursour, distanceX + 10 , 0)}
          ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.y)}`;
          xMultiplier = xMultiplier * -1;
      }
      if (stopY >= cursour.y) {
        distanceY = stopY - cursour.y;
      } else {
        distanceY = cursour.y - stopY;
      }
      points += `
        ${this.getLine(cursour, 0, ((distanceY - roundness / 2)) * yMultiplier)}
        ${this.getCorner(cursour, roundness, yMultiplier, xMultiplier, axis.x)}`;
      distanceX = stopX - cursour.x;
      points += 
        `${this.getLine(cursour, distanceX, 0)}`;
      return points;
    },
    onDependencyLineClick(event, dependencyLine) {
      this.root.state.popupData = {
        taskId: dependencyLine.taskId,
        prevTaskId: dependencyLine.prevTaskId,
        dependencyLineEl: event.target,
        dependencyLineRect: event.target.getBoundingClientRect()
      };
    },

    getCorner(cursour, roundness, yMultiplier, xMultiplier, side) {
      let resultPoint = {};
      switch (side) {
        case axis.y:
          cursour.x += roundness * xMultiplier;
          resultPoint = {
            x: cursour.x,
            y: cursour.y + roundness * yMultiplier
          };
          break;
        case axis.x:
          cursour.y += roundness * yMultiplier;
          resultPoint = {
            x: cursour.x + roundness * xMultiplier,
            y: cursour.y 
          };
      }
      return `Q ${cursour.x},${cursour.y} ${resultPoint.x},${resultPoint.y}`;
    },

    moveCursor(cursour, x, y){
      cursour.x = x;
      cursour.y = y;
      return `M ${cursour.x} ${cursour.y}`;
    },

    getLine(cursour, moveX, moveY) {
      cursour.x += moveX;
      cursour.y += moveY; 
      return `L ${cursour.x} ${cursour.y}`;
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
              points: this.getPoints(item.previousTask, task.id, item.type),
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
