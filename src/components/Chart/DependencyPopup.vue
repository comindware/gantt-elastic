<template>
  <div
    v-if="info"
    class="gantt-elastic__chart-tasks-dependency-popup"
    :style="{
      ...this.styles,
    }"
    @click="(event) => event.stopPropagation()"
    ref="dependencyPopupEl"
  >
    <div
      class="gantt-elastic__chart-tasks-dependency-popup-arrow-up-block"
      :style="{
        ...root.style['dependency-popup-arrow-up-block'],
      }"
    ></div>
    <div
      class="gantt-elastic__chart-tasks-dependency-popup-arrow-up-border"
      :style="{
        ...root.style['dependency-popup-arrow-up-border'],
      }"
    ></div>
    <div
      v-if="root.state.options.locale.dependencyPopup"
      class="gantt-elastic__chart-tasks-dependency-popup-title"
      :style="{
        ...root.style['dependency-popup-title'],
      }"
    >
      {{ root.state.options.locale.dependencyPopup.workDependency || '' }}
    </div>
    <div
      class="gantt-elastic__chart-tasks-dependency-popup-info-container"
      :style="{
        ...root.style['dependency-popup-info-container'],
      }"
      v-if="info.viewData"
    >
      <div
        class="gantt-elastic__chart-tasks-dependency-popup-info-row"
        :style="{
          ...root.style['dependency-popup-info-row'],
        }"
        v-for="(viewData, index) in info.viewData"
        :key="index"
      >
        <div
          class="gantt-elastic__chart-tasks-dependency-popup-info-title"
          :style="{
            ...root.style['dependency-popup-info-title'],
          }"
        >
          {{ viewData.title }}
        </div>
        <div
          class="gantt-elastic__chart-tasks-dependency-popup-info-value"
          :style="{
            ...root.style['dependency-popup-info-value'],
          }"
        >
          {{ viewData.value }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const TOP_OFFSET = 25;

export default {
  name: 'DependencyPopup',
  inject: ['root'],
  data() {
    return {
      styles: {},
    };
  },
  computed: {
    /**
     * Get data about selected dependency
     *
     * @returns {object}
     */
    info() {
      const taskId = this.root.state.popupData.taskId;
      const prevTaskId = this.root.state.popupData.prevTaskId;
      const localization = this.root.state.options.locale.dependencyPopup;
      const dependencyLineEl = this.root.state.popupData.dependencyLineEl;
      if (!taskId || !prevTaskId || !localization) {
        return null;
      }
      const task = this.root.state.tasks.find((item) => item.id === taskId);
      const prevTask = this.root.state.tasks.find((item) => item.id === prevTaskId);
      const deependencyInfo = task.dependentOn.find((item) => item.previousTask === prevTask.id);
      return {
        viewData: [
          {
            title: localization.predecessorWork,
            value: prevTask.title,
          },
          {
            title: localization.successorWork,
            value: task.title,
          },
          {
            title: localization.linkType,
            value: deependencyInfo.type,
          },
          {
            title: localization.slippage,
            value: this.getFormatedDuration(deependencyInfo.slippage),
          },
        ],
        dependencyLineEl,
      };
    },
  },
  watch: {
    /**
     * watcher to set styles object for popup
     */
    info() {
      const defaultStyles = this.root.style['chart-tasks-dependency-popup'];
      if (this.info == null) {
        this.styles = {
          ...defaultStyles,
        };
        return;
      }
      const popupWidth = this.root.state.options.dependencyPopup.width;
      const depLineRect = this.info.dependencyLineEl.getClientRects()[0];
      const left = depLineRect.left + depLineRect.width / 2 - popupWidth / 2 + 'px';
      const top = depLineRect.bottom + TOP_OFFSET + 'px';

      this.styles = {
        left,
        top,
        width: popupWidth + 'px',
        ...defaultStyles,
      };
    },
  },
  methods: {
    /**
     * @description Gets formated and localized duration string from the string of defined format
     *
     * @param {any} durationStr
     * @returns {string} Formated and localized duration string (both formats defines by options)
     */
    getFormatedDuration(durationStr) {
      const popupLocalization = this.root.state.options.locale.dependencyPopup;
      const regexesAndLocalePropNames = this.root.state.options.meta.durationFormat;
      let formatedDuration = '';
      if (popupLocalization && popupLocalization.duration && regexesAndLocalePropNames) {
        const replaceLocalizationParams = this.root.state.options.locale.replaceParams;
        regexesAndLocalePropNames.forEach((item) => {
          const matched = durationStr.match(new RegExp(item.regexStr));
          if (matched && matched.length) {
            formatedDuration += replaceLocalizationParams(popupLocalization.duration[item.localePropName], matched[0]);
          }
        });
      }
      return formatedDuration;
    },
  },
};
</script>