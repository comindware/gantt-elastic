<!--
/**
 * @fileoverview CalendarRow component
 * @license MIT
 * @author Rafal Pospiech <neuronet.io@gmail.com>
 * @package GanttElastic
 */
-->
<template>
  <div :class="'gantt-elastic__calendar-row gantt-elastic__calendar-row--' + which" :style="rowStyle">
    <div
      v-for="(item, itemIndex) in items"
      :key="item.key"
      :class="'gantt-elastic__calendar-row-rect gantt-elastic__calendar-row-rect--' + which"
      :style="rectStyle"
    >
      <div
        :class="'gantt-elastic__calendar-row-rect-child gantt-elastic__calendar-row-rect-child--' + which"
        v-for="(child, childIndex) in item.children"
        :key="child.key"
        :style="rectChildStyle[itemIndex][childIndex]"
      >
        <div
          :class="'gantt-elastic__calendar-row-text gantt-elastic__calendar-row-text--' + which"
        >
          {{ child.label }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CalendarRow',
  inject: ['root'],
  props: ['items', 'which'],
  data() {
    return {};
  },

  computed: {
    rowStyle() {
      return { ...this.root.style['calendar-row'], ...this.root.style['calendar-row--' + this.which] };
    },
    rectStyle() {
      return { ...this.root.style['calendar-row-rect'], ...this.root.style['calendar-row-rect--' + this.which] };
    },
    rectChildStyle() {
      const basicStyle = {
        ...this.root.style['calendar-row-rect-child'],
        ...this.root.style['calendar-row-rect-child--' + this.which]
      };
      const style = [];
      for (let item of this.items) {
        const childrenStyle = [];
        for (let child of item.children) {
          childrenStyle.push({
            ...basicStyle,
            width: child.width + 'px',
            height: child.height + 'px'
          });
        }
        style.push(childrenStyle);
      }
      return style;
    },
  }
};
</script>
