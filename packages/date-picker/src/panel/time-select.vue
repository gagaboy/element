<template>
  <transition name="el-zoom-in-top" @before-enter="handleMenuEnter" @after-leave="$emit('dodestroy')">
    <div
      ref="popper"
      v-show="visible"
      :style="{ width: width + 'px' }"
      :class="popperClass"
      class="el-picker-panel time-select">
      <el-scrollbar noresize wrap-class="el-picker-panel__content">
        <div class="time-select-item"
          v-for="item in items"
          :class="{ selected: value === item.value, disabled: item.disabled }"
          :disabled="item.disabled"
          @click="handleClick(item)">{{ item.value }}</div>
      </el-scrollbar>
    </div>
  </transition>
</template>

<script type="text/babel">
  import ElScrollbar from 'element-ui/packages/scrollbar';
  import scrollIntoView from 'element-ui/src/utils/scroll-into-view';

  const parseTime = function(time) {
    const values = (time || '').split(':');
    if (values.length >= 2) {
      const hours = parseInt(values[0], 10);
      const minutes = parseInt(values[1], 10);

      return {
        hours,
        minutes
      };
    }
    /* istanbul ignore next */
    return null;
  };

  const compareTime = function(time1, time2) {
    const value1 = parseTime(time1);
    const value2 = parseTime(time2);

    const minutes1 = value1.minutes + value1.hours * 60;
    const minutes2 = value2.minutes + value2.hours * 60;

    if (minutes1 === minutes2) {
      return 0;
    }

    return minutes1 > minutes2 ? 1 : -1;
  };

  const formatTime = function(time) {
    return (time.hours < 10 ? '0' + time.hours : time.hours) + ':' + (time.minutes < 10 ? '0' + time.minutes : time.minutes);
  };

  const nextTime = function(time, step) {
    const timeValue = parseTime(time);
    const stepValue = parseTime(step);

    const next = {
      hours: timeValue.hours,
      minutes: timeValue.minutes
    };

    next.minutes += stepValue.minutes;
    next.hours += stepValue.hours;

    next.hours += Math.floor(next.minutes / 60);
    next.minutes = next.minutes % 60;

    return formatTime(next);
  };

  export default {
    components: { ElScrollbar },

    watch: {
      value(val) {
        if (!val) return;
        if (this.minTime && compareTime(val, this.minTime) < 0) {
          this.$emit('pick');
        } else if (this.maxTime && compareTime(val, this.maxTime) > 0) {
          this.$emit('pick');
        }
        this.$nextTick(() => this.scrollToOption());
      }
    },

    methods: {
      handleClick(item) {
        if (!item.disabled) {
          this.$emit('pick', item.value);
        }
      },

      handleClear() {
        this.$emit('pick');
      },

      scrollToOption(className = 'selected') {
        const menu = this.$refs.popper.querySelector('.el-picker-panel__content');
        scrollIntoView(menu, menu.getElementsByClassName(className)[0]);
      },

      handleMenuEnter() {
        this.$nextTick(() => this.scrollToOption());
      },

      scrollDown(step) {
        const items = this.items;
        let index = items.map(item => item.value).indexOf(this.value);
        let length = items.length;
        let total = Math.abs(step);
        step = step > 0 ? 1 : -1;
        while (length-- && total) {
          index = (index + step + items.length) % items.length;
          const item = items[index];
          if (!item.disabled) {
            total--;
          }
        }
        if (!items[index].disabled) {
          this.value = items[index].value;
        }
      }
    },

    data() {
      return {
        popperClass: '',
        start: '09:00',
        end: '18:00',
        step: '00:30',
        value: '',
        visible: false,
        minTime: '',
        maxTime: '',
        width: 0
      };
    },

    computed: {
      items() {
        const start = this.start;
        const end = this.end;
        const step = this.step;

        const result = [];

        if (start && end && step) {
          let current = start;
          while (compareTime(current, end) <= 0) {
            result.push({
              value: current,
              disabled: compareTime(current, this.minTime || '-1:-1') <= 0 ||
                compareTime(current, this.maxTime || '100:100') >= 0
            });
            current = nextTime(current, step);
          }
        }

        return result;
      }
    }
  };
</script>
