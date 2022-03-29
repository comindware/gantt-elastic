<!--
/**
 * @fileoverview Calendar component
 * @license MIT
 * @author Rafal Pospiech <neuronet.io@gmail.com>
 * @package GanttElastic
 */
-->
<template>
  <div
    class="gantt-elastic__calendar-wrapper"
    :style="{ ...root.style['calendar-wrapper'], width: width + 'px'}"
  >
    <div class="gantt-elastic__calendar" :style="{ ...root.style['calendar'], width: width + 'px'}">
      <calendar-row :items="dates.decades" which="decade" v-if="calendar.decade.display"></calendar-row>
      <calendar-row :items="dates.years" which="year" v-if="calendar.year.display"></calendar-row>
      <calendar-row :items="dates.quarters" which="quarter" v-if="calendar.quarter.display"></calendar-row>
      <calendar-row :items="dates.months" which="month" v-if="calendar.month.display"></calendar-row>
      <calendar-row :items="dates.weeks" which="week" v-if="calendar.week.display"></calendar-row>
      <calendar-row :items="dates.days" which="day" v-if="calendar.day.display"></calendar-row>
      <calendar-row :items="dates.hours" which="hour" v-if="calendar.hour.display"></calendar-row>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs';
import CalendarRow from './CalendarRow.vue';

export default {
  name: 'Calendar',
  components: {
    CalendarRow
  },
  inject: ['root'],
  props: ['calendar', 'times', 'locale', 'width'],
  data() {
    return {};
  },

  methods: {
    /**
     * How many hours will fit?
     *
     * @returns {object}
     */
    howManyHoursFit(dayIndex) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullCellWidth = this.times.steps[dayIndex].width.px;
      let formatNames = Object.keys(this.calendar.hour.format);
      for (let hours = 24; hours > 1; hours = Math.ceil(hours / 2)) {
        for (let formatName of formatNames) {
          if (
            (this.calendar.hour.maxWidths[formatName] + additionalSpace) * hours <= fullCellWidth &&
            hours > 1
          ) {
            return {
              count: hours,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: ''
      };
    },

    /**
     * How many days will fit?
     *
     * @returns {object}
     */
    howManyDaysFit(day) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullWidth = this.width;
      let formatNames = Object.keys(day.format);
      for (let days = this.times.steps.length; days > 1; days = Math.ceil(days / 2)) {
        for (let formatName of formatNames) {
          if (
            (day.maxWidths[formatName] + additionalSpace) * days <= fullWidth &&
            days > 1
          ) {
            return {
              count: days,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: ''
      };
    },

    /**
     * How many weeks will fit?
     *
     * @returns {object}
     */
    howManyWeeksFit(week, firstTime, lastTime) {
        const stroke = 1;
        const additionalSpace = stroke + 2;
        let fullWidth = this.width;
        let formatNames = Object.keys(week.format);
        let weeksCount = this.root.weeksCount(firstTime,lastTime);
        if (weeksCount === 1) {
          for (let formatName of formatNames) {
            if (week.maxWidths[formatName] + additionalSpace <= fullWidth) {
              return {
                count: 1,
                type: formatName
              };
            }
          }
        }
        for (let weeks = weeksCount; weeks > 1; weeks = Math.ceil(weeks / 2)) {
          for (let formatName of formatNames) {
            if (
              (week.maxWidths[formatName] + additionalSpace) * weeks <= fullWidth &&
              weeks > 1
            ) {
              return {
                count: weeks,
                type: formatName
              };
            }
          }
        }
        return {
          count: 0,
          type: formatNames[0]
        };
      },

    /**
     * How many months will fit?
     *
     * @returns {object}
     */
    howManyMonthsFit(month, firstTime, lastTime) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullWidth = this.width;
      let formatNames = Object.keys(month.format);
      let monthsCount = this.root.monthsCount(firstTime, lastTime);
      if (monthsCount === 1) {
        for (let formatName of formatNames) {
          if (month.maxWidths[formatName] + additionalSpace <= fullWidth) {
            return {
              count: 1,
              type: formatName
            };
          }
        }
      }
      for (let months = monthsCount; months > 1; months = Math.ceil(months / 2)) {
        for (let formatName of formatNames) {
          if (
            (month.maxWidths[formatName] + additionalSpace) * months <= fullWidth &&
            months > 1
          ) {
            return {
              count: months,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: formatNames[0]
      };
    },

    /**
     * How many quarters will fit?
     *
     * @returns {object}
     */
    howManyQuartersFit(quarter, firstTime, lastTime) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullWidth = this.width;
      let formatNames = Object.keys(quarter.format);
      let quartersCount = this.root.quartersCount(firstTime, lastTime);
      if (quartersCount === 1) {
        for (let formatName of formatNames) {
          if (quarter.maxWidths[formatName] + additionalSpace <= fullWidth) {
            return {
              count: 1,
              type: formatName
            };
          }
        }
      }
      for (let quarters = quartersCount; quarters > 1; quarters = Math.ceil(quarters / 2)) {
        for (let formatName of formatNames) {
          if (
            (quarter.maxWidths[formatName] + additionalSpace) * quarters <= fullWidth &&
            quarters > 1
          ) {
            return {
              count: quarters,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: formatNames[0]
      };
    },

    /**
     * How many years will fit?
     *
     * @returns {object}
     */
    howManyYearsFit(year, firstTime, lastTime) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullWidth = this.width;
      let formatNames = Object.keys(year.format);
      let yearsCount = this.root.yearsCount(firstTime, lastTime);
      if (yearsCount === 1) {
        for (let formatName of formatNames) {
          if (year.maxWidths[formatName] + additionalSpace <= fullWidth) {
            return {
              count: 1,
              type: formatName
            };
          }
        }
      }
      for (let years = yearsCount; years > 1; years = Math.ceil(years / 2)) {
        for (let formatName of formatNames) {
          if (
            (year.maxWidths[formatName] + additionalSpace) * years <= fullWidth &&
            years > 1
          ) {
            return {
              count: years,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: formatNames[0]
      };
    },

    howManyDecadesFit(decade, firstTime, lastTime) {
      const stroke = 1;
      const additionalSpace = stroke + 2;
      let fullWidth = this.width;
      let formatNames = Object.keys(decade.format);
      let decadesCount = this.root.decadesCount(firstTime, lastTime);
      if (decadesCount === 1) {
        for (let formatName of formatNames) {
          if (decade.maxWidths[formatName] + additionalSpace <= fullWidth) {
            return {
              count: 1,
              type: formatName
            };
          }
        }
      }
      for (let decades = decadesCount; decades > 1; decades = Math.ceil(decades / 2)) {
        for (let formatName of formatNames) {
          if (
            (decade.maxWidths[formatName] + additionalSpace) * decades <= fullWidth &&
            decades > 1
          ) {
            return {
              count: decades,
              type: formatName
            };
          }
        }
      }
      return {
        count: 0,
        type: formatNames[0]
      };
    },

    /**
     * Generate hours
     *
     * @returns {array}
     */
    generateHours() {
      let allHours = [];
      if (!this.calendar.hour.display) {
        return allHours;
      }
      const steps = this.times.steps;
      for (let hourIndex = 0, len = steps.length; hourIndex < len; hourIndex++) {
        const hoursCount = this.howManyHoursFit(hourIndex);
        if (hoursCount.count === 0) {
          continue;
        }
        const hours = { key: hourIndex + 'step', children: [] };
        const hourStep = 24 / hoursCount.count;
        const hourWidthPx = steps[hourIndex].width.px / hoursCount.count;
        for (let i = 0, len = hoursCount.count; i < len; i++) {
          const hour = i * hourStep;
          let index = hourIndex;
          if (hourIndex > 0) {
            index = hourIndex - Math.floor(hourIndex / 24) * 24;
          }
          let textWidth = 0;
          if (typeof this.calendar.hour.widths[index] !== 'undefined') {
            textWidth = this.calendar.hour.widths[index][hoursCount.type];
          }
          let x = steps[hourIndex].offset.px + hourWidthPx * i;
          hours.children.push({
            index: hourIndex,
            key: 'h' + i,
            x,
            y: this.calendar.day.height + this.calendar.month.height,
            width: hourWidthPx,
            textWidth,
            height: this.calendar.hour.height,
            label: this.calendar.hour.formatted[hoursCount.type][hour]
          });
        }
        allHours.push(hours);
      }
      return allHours;
    },

    /**
     * Generate days
     *
     * @returns {array}
     */
    generateDays() {
      const day = this.calendar.day;
      let days = [];
      if (!day.display) {
        return days;
      }
      const daysCount = this.howManyDaysFit(day);
      if (daysCount.count === 0) {
        return days;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      const dayStep = Math.ceil(steps.length / daysCount.count);
      for (let dayIndex = 0, len = steps.length; dayIndex < len; dayIndex += dayStep) {
        let dayWidthPx = 0;
        // day could be shorter (daylight saving time) so join widths and divide
        for (let currentStep = 0; currentStep < dayStep; currentStep++) {
          if (typeof steps[dayIndex + currentStep] !== 'undefined') {
            dayWidthPx += steps[dayIndex + currentStep].width.px;
          }
        }
        const date = dayjs(steps[dayIndex].time);
        let textWidth = 0;
        if (typeof day.widths[dayIndex] !== 'undefined') {
          textWidth = day.widths[dayIndex][daysCount.type];
        }
        let x = steps[dayIndex].offset.px;
        const label = day.format[daysCount.type](date.locale(localeName));
        days.push({
          index: dayIndex,
          key: steps[dayIndex].time + 'd',
          x,
          y: this.calendar.month.height,
          width: dayWidthPx,
          textWidth,
          height: day.height,
          label
        });
      }
      return days.map(item => ({
        key: item.key,
        children: [item]
      }));
    },

    /**
     * Generate weeks
     *
     * @returns {array}
     */
    generateWeeks(firstTime, lastTime) {
      let weeks = [];
      const week = this.calendar.week;
      if (!week.display) {
        return weeks;
      }
      const weeksCount = this.howManyWeeksFit(week, firstTime, lastTime);
      if (weeksCount.count === 0) {
        return weeks;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      let formatNames = Object.keys(week.format);
      let currentDate = dayjs(firstTime);
      for (let weekIndex = 0; weekIndex < weeksCount.count; weekIndex++) {
        let weekWidth = 0;
        let weekOffset = Number.MAX_SAFE_INTEGER;
        let finalDate = dayjs(currentDate)
          .add(1, 'week')
          .startOf('week');
        if (finalDate.valueOf() > lastTime) {
          finalDate = dayjs(lastTime);
        }
        // we must find first and last step to get the offsets / widths
        for (let step = 0, len = steps.length; step < len; step++) {
          let currentStep = steps[step];
          if (currentStep.time >= currentDate.valueOf() && currentStep.time < finalDate.valueOf()) {
            weekWidth += currentStep.width.px;
            if (currentStep.offset.px < weekOffset) {
              weekOffset = currentStep.offset.px;
            }
          }
        }
        let label = '';
        let choosenFormatName;
        for (let formatName of formatNames) {
          if (week.maxWidths[formatName] + 2 <= weekWidth) {
            const formatFunction = week.format[formatName];
            const dateLocal = currentDate.locale(localeName);
            label = formatFunction(dateLocal);
            if (formatName === 'long') {
                const endWeek = currentDate.endOf('week');
                label = `${label} - ${formatFunction(endWeek)} ${endWeek.format('YYYY')}`.toLowerCase();
            }
            choosenFormatName = formatName;
          }
        }
        let textWidth = 0;
        if (typeof week.widths[weekIndex] !== 'undefined') {
          textWidth = week.widths[weekIndex][choosenFormatName];
        }
        let x = weekOffset;
        weeks.push({
          index: weekIndex,
          key: weekIndex + 'm',
          x,
          y: 0,
          width: weekWidth,
          textWidth,
          choosenFormatName,
          height: week.height,
          label
        });
        currentDate = currentDate.add(1, 'week').startOf('week');
        if (currentDate.valueOf() > lastTime) {
          currentDate = dayjs(lastTime);
        }
      }
      return weeks.map(item => ({
        key: item.key,
        children: [item]
      }));
    },

    /**
     * Generate months
     *
     * @returns {array}
     */
    generateMonths(firstTime, lastTime) {
      let months = [];
      const month = this.calendar.month;
      if (!month.display) {
        return months;
      }
      const monthsCount = this.howManyMonthsFit(month, firstTime, lastTime);
      if (monthsCount.count === 0) {
        return months;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      let formatNames = Object.keys(month.format);
      let currentDate = dayjs(firstTime);
      for (let monthIndex = 0; monthIndex < monthsCount.count; monthIndex++) {
        let monthWidth = 0;
        let monthOffset = Number.MAX_SAFE_INTEGER;
        let finalDate = dayjs(currentDate)
          .add(1, 'month')
          .startOf('month');
        if (finalDate.valueOf() > lastTime) {
          finalDate = dayjs(lastTime);
        }
        // we must find first and last step to get the offsets / widths
        for (let step = 0, len = steps.length; step < len; step++) {
          let currentStep = steps[step];
          if (currentStep.time >= currentDate.valueOf() && currentStep.time < finalDate.valueOf()) {
            monthWidth += currentStep.width.px;
            if (currentStep.offset.px < monthOffset) {
              monthOffset = currentStep.offset.px;
            }
          }
        }
        let label = '';
        let choosenFormatName;
        for (let formatName of formatNames) {
          if (month.maxWidths[formatName] + 2 <= monthWidth) {
            label = month.format[formatName](currentDate.locale(localeName));
            choosenFormatName = formatName;
          }
        }
        let textWidth = 0;
        if (typeof month.widths[monthIndex] !== 'undefined') {
          textWidth = month.widths[monthIndex][choosenFormatName];
        }
        let x = monthOffset;
        months.push({
          index: monthIndex,
          key: monthIndex + 'm',
          x,
          y: 0,
          width: monthWidth,
          textWidth,
          choosenFormatName,
          height: month.height,
          label
        });
        currentDate = currentDate.add(1, 'month').startOf('month');
        if (currentDate.valueOf() > lastTime) {
          currentDate = dayjs(lastTime);
        }
      }
      return months.map(item => ({
        key: item.key,
        children: [item]
      }));
    },

    /**
     * Generate quarters
     *
     * @returns {array}
     */
    generateQuarters(firstTime, lastTime) {
      let quarters = [];
      const quarter = this.calendar.quarter;
      if (!quarter.display) {
        return quarters;
      }
      const quartersCount = this.howManyQuartersFit(quarter, firstTime, lastTime);
      if (quartersCount.count === 0) {
        return quarters;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      let formatNames = Object.keys(quarter.format);
      let currentDate = dayjs(firstTime);
      for (let quarterIndex = 0; quarterIndex < quartersCount.count; quarterIndex++) {
        let quarterWidth = 0;
        let quarterOffset = Number.MAX_SAFE_INTEGER;
        let finalDate = dayjs(currentDate)
          .add(1, 'quarter')
          .startOf('quarter');
        if (finalDate.valueOf() > lastTime) {
          finalDate = dayjs(lastTime);
        }
        // we must find first and last step to get the offsets / widths
        for (let step = 0, len = steps.length; step < len; step++) {
          let currentStep = steps[step];
          if (currentStep.time >= currentDate.valueOf() && currentStep.time < finalDate.valueOf()) {
            quarterWidth += currentStep.width.px;
            if (currentStep.offset.px < quarterOffset) {
              quarterOffset = currentStep.offset.px;
            }
          }
        }
        let label = '';
        let choosenFormatName;
        for (let formatName of formatNames) {
          if (quarter.maxWidths[formatName] + 2 <= quarterWidth) {
            const format = quarter.format[formatName];
            const day = currentDate.locale(localeName);
            label = format(day);
            choosenFormatName = formatName;
          }
        }
        let textWidth = 0;
        if (typeof quarter.widths[quarterIndex] !== 'undefined') {
          textWidth = quarter.widths[quarterIndex][choosenFormatName];
        }
        let x = quarterOffset;
        quarters.push({
          index: quarterIndex,
          key: quarterIndex + 'm',
          x,
          y: 0,
          width: quarterWidth,
          textWidth,
          choosenFormatName,
          height: quarter.height,
          label
        });
        currentDate = currentDate.add(1, 'quarter').startOf('quarter');
        if (currentDate.valueOf() > lastTime) {
          currentDate = dayjs(lastTime);
        }
      }
      return quarters.map(item => ({
        key: item.key,
        children: [item]
      }));
    },

    /**
     * Generate years
     *
     * @returns {array}
     */
    generateYears(firstTime, lastTime) {
      let years = [];
      const year = this.calendar.year;
      if (!year.display) {
        return years;
      }
      const yearsCount = this.howManyYearsFit(year, firstTime, lastTime);
      if (yearsCount.count === 0) {
        return years;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      let formatNames = Object.keys(year.format);
      let currentDate = dayjs(firstTime);
      for (let yearIndex = 0; yearIndex < yearsCount.count; yearIndex++) {
        let yearWidth = 0;
        let yearOffset = Number.MAX_SAFE_INTEGER;
        let finalDate = dayjs(currentDate)
          .add(1, 'year')
          .startOf('year');
        if (finalDate.valueOf() > lastTime) {
          finalDate = dayjs(lastTime);
        }
        // we must find first and last step to get the offsets / widths
        for (let step = 0, len = steps.length; step < len; step++) {
          let currentStep = steps[step];
          if (currentStep.time >= currentDate.valueOf() && currentStep.time < finalDate.valueOf()) {
            yearWidth += currentStep.width.px;
            if (currentStep.offset.px < yearOffset) {
              yearOffset = currentStep.offset.px;
            }
          }
        }
        let label = '';
        let choosenFormatName;
        for (let formatName of formatNames) {
          if (year.maxWidths[formatName] + 2 <= yearWidth) {
            label = year.format[formatName](currentDate.locale(localeName));
            choosenFormatName = formatName;
          }
        }
        let textWidth = 0;
        if (typeof year.widths[yearIndex] !== 'undefined') {
          textWidth = year.widths[yearIndex][choosenFormatName];
        }
        let x = yearOffset;
        years.push({
          index: yearIndex,
          key: yearIndex + 'm',
          x,
          y: 0,
          width: yearWidth,
          textWidth,
          choosenFormatName,
          height: year.height,
          label
        });
        currentDate = currentDate.add(1, 'year').startOf('year');
        if (currentDate.valueOf() > lastTime) {
          currentDate = dayjs(lastTime);
        }
      }
      return years.map(item => ({
        key: item.key,
        children: [item]
      }));
    },

    generateDecade(firstTime, lastTime) {
      let decades = [];
      const decade = this.calendar.decade;
      if (!decade.display) {
        return decades;
      }
      const decadesCount = this.howManyDecadesFit(decade, firstTime, lastTime);
      if (decadesCount.count === 0) {
        return decades;
      }
      const steps = this.times.steps;
      const localeName = this.locale.name;
      let formatNames = Object.keys(decade.format);
      let currentDate = dayjs(firstTime);
      for (let decadeIndex = 0; decadeIndex < decadesCount.count; decadeIndex++) {
        let decadeWidth = 0;
        let decadeOffset = Number.MAX_SAFE_INTEGER;
        let finalDate = this.__addDecade(dayjs(currentDate));
        if (finalDate.valueOf() > lastTime) {
          finalDate = dayjs(lastTime);
        }
        for (let step = 0, len = steps.length; step < len; step++) {
          let currentStep = steps[step];
          if (currentStep.time >= currentDate.valueOf() && currentStep.time < finalDate.valueOf()) {
            decadeWidth += currentStep.width.px;
            if (currentStep.offset.px < decadeOffset) {
              decadeOffset = currentStep.offset.px;
            }
          }
        }
        let label = '';
        let choosenFormatName;
        for (let formatName of formatNames) {
          if (decade.maxWidths[formatName] + 2 <= decadeWidth) {
            label = decade.format[formatName](currentDate.locale(localeName));
            label = label + ' - ' + decade.format[formatName](finalDate.subtract(1, 'year').locale(localeName));
            choosenFormatName = formatName;
          }
        }
        let textWidth = 0;
        if (typeof decade.widths[decadeIndex] !== 'undefined') {
          textWidth = decade.widths[decadeIndex][choosenFormatName];
        }
        let x = decadeOffset;
        decades.push({
          index: decadeIndex,
          key: decadeIndex + 'm',
          x,
          y: 0,
          width: decadeWidth,
          textWidth,
          choosenFormatName,
          height: decade.height,
          label
        });
        currentDate = this.__addDecade(currentDate);
        if (currentDate.valueOf() > lastTime) {
          currentDate = dayjs(lastTime);
        }
      }
      return decades.map(item => ({
        key: item.key,
        children: [item]
      }));
    },
    
    __addDecade(currentDate) {
      return currentDate.add(10, 'year').startOf('year');
    },

    /**
     * Sum all calendar rows height and return result
     *
     * @returns {int}
     */
    calculateCalendarDimensions({ hours, days, weeks, months, quarters, years, decades }) {
      let height = 0;
      if (this.calendar.hour.display && hours.length > 0) {
        height += this.calendar.hour.height;
      }
      if (this.calendar.day.display && days.length > 0) {
        height += this.calendar.day.height;
      }
      if (this.calendar.week.display && weeks.length > 0) {
        height += this.calendar.week.height;
      }
      if (this.calendar.month.display && months.length > 0) {
        height += this.calendar.month.height;
      }
      if (this.calendar.quarter.display && quarters.length > 0) {
        height += this.calendar.quarter.height;
      }
      if (this.calendar.year.display && years.length > 0) {
        height += this.calendar.year.height;
      }
      if (this.calendar.decade.display && decades.length > 0) {
        height += this.calendar.decade.height;
      }
      if (height === 0) {
        height += this.calendar.defaultHeight;
      }
      this.calendar.height = height;
    }
  },

  computed: {
    dates() {
      const firstTime = this.times.firstTime;
      const lastTime = this.times.lastTime;
      const hours = this.generateHours();
      const days = this.generateDays();
      const weeks = this.generateWeeks(firstTime, lastTime);
      const months = this.generateMonths(firstTime, lastTime);
      const quarters = this.generateQuarters(firstTime, lastTime);
      const years = this.generateYears(firstTime, lastTime);
      const decades = this.generateDecade(firstTime, lastTime);
      const allDates = { hours, days, weeks, months, quarters, years, decades };
      this.calculateCalendarDimensions(allDates);
      return allDates;
    }
  }
};
</script>
