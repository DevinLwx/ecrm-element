<template>
  <div class="el-time-spinner" :class="{ 'has-seconds': showSeconds }">
    <template v-if="!arrowControl">
      <el-scrollbar
        @mouseenter.native="emitSelectRange('hours')"
        @mousemove.native="adjustCurrentSpinner('hours')"
        class="el-time-spinner__wrapper"
        wrap-style="max-height: inherit;"
        view-class="el-time-spinner__list"
        noresize
        tag="ul"
        ref="hours">
        <li
          @click="handleClick('hours', { value: hour % 24, disabled: disabled })"
          v-for="(disabled, hour) in hoursList"
          class="el-time-spinner__item"
          :key="hour"
          :class="{ 'active': hour === hours || hour % 24 === hours, 'disabled': disabled }">
          {{ ('0' + (amPmMode ? (hour % 12 || 12) : hour ) % 24).slice(-2) }}{{ amPm(hour) }}</li>
      </el-scrollbar>
      <el-scrollbar
        @mouseenter.native="emitSelectRange('minutes')"
        @mousemove.native="adjustCurrentSpinner('minutes')"
        class="el-time-spinner__wrapper"
        wrap-style="max-height: inherit;"
        view-class="el-time-spinner__list"
        noresize
        tag="ul"
        ref="minutes">
        <li
          @click="handleClick('minutes', { value: key % 60, disabled: false })"
          v-for="(enabled, key) in minutesList"
          :key="key"
          v-if="enabled"
          class="el-time-spinner__item"
          :class="{ 'active': key === minutes || key % 60 === minutes, disabled: !enabled }">{{ ('0' + key % 60).slice(-2) }}</li>
      </el-scrollbar>
      <el-scrollbar
        v-show="showSeconds"
        @mouseenter.native="emitSelectRange('seconds')"
        @mousemove.native="adjustCurrentSpinner('seconds')"
        class="el-time-spinner__wrapper"
        wrap-style="max-height: inherit;"
        view-class="el-time-spinner__list"
        noresize
        tag="ul"
        ref="seconds">
        <li
          @click="handleClick('seconds', { value: key % 60, disabled: false })"
          v-for="(second, key) in 120"
          class="el-time-spinner__item"
          :class="{ 'active': key === seconds || key % 60 === seconds}"
          :key="key">{{ ('0' + key % 60).slice(-2) }}</li>
      </el-scrollbar>
    </template>
    <template v-if="arrowControl">
      <div
        @mouseenter="emitSelectRange('hours')"
        class="el-time-spinner__wrapper is-arrow">
        <i v-repeat-click="decrease" class="el-time-spinner__arrow el-icon-arrow-up"></i>
        <i v-repeat-click="increase" class="el-time-spinner__arrow el-icon-arrow-down"></i>
        <ul class="el-time-spinner__list" ref="hours">
          <li
            class="el-time-spinner__item"
            :class="{ 'active': hour === hours || hour % 24 === hours, 'disabled': hoursList[hour] }"
            v-for="(hour, key) in arrowHourList"
            :key="key">{{ hour === undefined ? '' : ('0' + (amPmMode ? (hour % 12 || 12) : hour ) % 24).slice(-2) + amPm(hour) }}</li>
        </ul>
      </div>
      <div
        @mouseenter="emitSelectRange('minutes')"
        class="el-time-spinner__wrapper is-arrow">
        <i v-repeat-click="decrease" class="el-time-spinner__arrow el-icon-arrow-up"></i>
        <i v-repeat-click="increase" class="el-time-spinner__arrow el-icon-arrow-down"></i>
        <ul class="el-time-spinner__list" ref="minutes">
          <li
            class="el-time-spinner__item"
            :class="{ 'active': minute === minutes || minute % 60 === minutes }"
            v-for="(minute, key) in arrowMinuteList"
            :key="key">
            {{ minute === undefined ? '' : ('0' + minute % 60).slice(-2) }}
          </li>
        </ul>
      </div>
      <div
        @mouseenter="emitSelectRange('seconds')"
        class="el-time-spinner__wrapper is-arrow"
        v-if="showSeconds">
        <i v-repeat-click="decrease" class="el-time-spinner__arrow el-icon-arrow-up"></i>
        <i v-repeat-click="increase" class="el-time-spinner__arrow el-icon-arrow-down"></i>
        <ul class="el-time-spinner__list" ref="seconds">
          <li
            v-for="(second, key) in arrowSecondList"
            class="el-time-spinner__item"
            :class="{ 'active': second === seconds || second % 60 === seconds }"
            :key="key">
            {{ second === undefined ? '' : ('0' + second % 60).slice(-2) }}
          </li>
        </ul>
      </div>
    </template>
  </div>
</template>

<script type="text/babel">
  import { getRangeHours, getRangeMinutes, modifyTime } from 'element-ui/src/utils/date-util';
  import ElScrollbar from 'element-ui/packages/scrollbar';
  import RepeatClick from 'element-ui/src/directives/repeat-click';

  export default {
    components: { ElScrollbar },

    directives: {
      repeatClick: RepeatClick
    },

    props: {
      date: {},
      defaultValue: {}, // reserved for future use
      showSeconds: {
        type: Boolean,
        default: true
      },
      arrowControl: Boolean,
      amPmMode: {
        type: String,
        default: '' // 'a': am/pm; 'A': AM/PM
      },
      minutesInterval: {
        type: [String, Number],
        default: 1
      }
    },

    computed: {
      hours() {
        return this.date.getHours();
      },
      minutes() {
        return this.date.getMinutes() - this.date.getMinutes() % this.minutesInterval;
      },
      seconds() {
        return this.date.getSeconds();
      },
      hoursList() {
        const list = getRangeHours(this.selectableRange);
        return list.concat(list);
      },
      minutesList() {
        const list = getRangeMinutes(this.selectableRange, this.hours, this.minutesInterval);
        return list.concat(list);
      },
      arrowHourList() {
        const hours = this.hours;
        const list = [
          hours > 0 ? hours - 1 : undefined,
          hours,
          hours < 23 ? hours + 1 : undefined
        ];
        return list.concat(list);
      },
      arrowMinuteList() {
        const minutes = this.minutes;
        const list = [
          minutes > 0 ? minutes - 1 : undefined,
          minutes,
          minutes < 59 ? minutes + 1 : undefined
        ];
        return list.concat(list);
      },
      arrowSecondList() {
        const seconds = this.seconds;
        const list = [
          seconds > 0 ? seconds - 1 : undefined,
          seconds,
          seconds < 59 ? seconds + 1 : undefined
        ];
        return list.concat(list);
      }
    },

    data() {
      return {
        selectableRange: [],
        currentScrollbar: null,
        beforeScrollTop: 0
      };
    },

    mounted() {
      this.$nextTick(() => {
        !this.arrowControl && this.bindScrollEvent();
      });
    },

    methods: {
      increase() {
        this.scrollDown(1);
      },

      decrease() {
        this.scrollDown(-1);
      },

      modifyDateField(type, value) {
        switch (type) {
          case 'hours': this.$emit('change', modifyTime(this.date, value, this.minutes, this.seconds)); break;
          case 'minutes': this.$emit('change', modifyTime(this.date, this.hours, value, this.seconds)); break;
          case 'seconds': this.$emit('change', modifyTime(this.date, this.hours, this.minutes, value)); break;
        }
      },

      handleClick(type, {value, disabled}) {
        if (!disabled) {
          this.modifyDateField(type, value);
          this.emitSelectRange(type);
          this.adjustSpinner(type, value);
          this.selectTimeEnd(type);
        }
      },

      selectTimeEnd(type) {
        if (type === 'minutes') {
          if (!this.showSeconds) {
            this.$emit('select-end');
          }
        } else if (type === 'seconds') {
          this.$emit('select-end');
        }
      },

      emitSelectRange(type) {
        if (type === 'hours') {
          this.$emit('select-range', 0, 2);
        } else if (type === 'minutes') {
          this.$emit('select-range', 3, 5);
        } else if (type === 'seconds') {
          this.$emit('select-range', 6, 8);
        }
        this.currentScrollbar = type;
      },

      bindScrollEvent() {
        const bindFuntion = (type, max) => {
          this.beforeScrollTop = this.$refs[type].wrap.scrollTop;
          this.$refs[type].wrap.onscroll = (e) => {
            let scrollType = 'down';
            if (this.beforeScrollTop > this.$refs[type].wrap.scrollTop) {
              scrollType = 'up';
            } else {
              scrollType = 'down';
            }
            // TODO: scroll is emitted when set scrollTop programatically
            // should find better solutions in the future!
            this.handleScroll(type, e, scrollType, max);
          };
        };
        bindFuntion('hours', 23);
        bindFuntion('minutes', 59);
        bindFuntion('seconds', 59);
      },

      scrollUpDown(type, scrollType) {
        if (scrollType === 'down') {
          this.beforeScrollTop = this.$refs[type].wrap.scrollTop + 1;
        } else {
          this.beforeScrollTop = this.$refs[type].wrap.scrollTop - 1;
        }
      },

      scrollOperation(type, e, scrollType, max) {
        const wrap = this.$refs[type].wrap;
        if (wrap.scrollTop % Math.ceil(wrap.scrollHeight / 2) === 0) { // 滚动一整个块后
          if (scrollType === 'up' && wrap.scrollTop === 0) { //  || this.beforeScrollTop === 0
            wrap.scrollTop = (wrap.scrollHeight - this.typeItemHeight(type)) / 2 + 4;
            this.beforeScrollTop = wrap.scrollTop + 1;
          }
          if (scrollType === 'down') {
            wrap.scrollTop += 4;
          }
          this.handleScroll(type, e, scrollType, max);
        } else {
          if (this.beforeScrollTop >= wrap.scrollHeight / 2) { // 判断滚动高度,当滚动高度大于area本身的高度时，使其回到原点重新滚动
            wrap.scrollTop = Math.ceil(this.beforeScrollTop - wrap.scrollHeight / 2);
          }

          this.scrollUpDown(type, scrollType);
        }
      },

      handleScroll(type, e, scrollType, max) {
        const value = Math.min(Math.floor((this.$refs[type].wrap.scrollTop - (this.scrollBarHeight(type) * 0.5 - 10) / this.typeItemHeight(type) + 3 + 80 + 16) / this.typeItemHeight(type)));
        this.scrollUpDown(type, scrollType);
        this.scrollOperation(type, e, scrollType, max);
        if (type === 'hours') {
          this.modifyDateField(type, value % (max + 1));
        } else if (type === 'minutes') {
          this.modifyDateField(type, value * this.minutesInterval % (max + 1));
        } else {
          this.modifyDateField(type, value % (max + 1));
        }
      },

      // NOTE: used by datetime / date-range panel
      //       renamed from adjustScrollTop
      //       should try to refactory it
      adjustSpinners() {
        this.adjustSpinner('hours', this.hours);
        this.adjustSpinner('minutes', this.minutes / this.minutesInterval);
        this.adjustSpinner('seconds', this.seconds);
      },

      adjustCurrentSpinner(type) {
        let value = this[type];
        if (type === 'minutes') {
          value = Math.ceil(this[type] / this.minutesInterval);
        }
        this.adjustSpinner(type, value);
      },

      adjustSpinner(type, value) {
        if (this.arrowControl) return;
        const el = this.$refs[type].wrap;
        if (el) {
          if (value * this.typeItemHeight(type) >= 80) {
            el.scrollTop = Math.max(0, value * this.typeItemHeight(type) - 80);
          } else {
            el.scrollTop = Math.max(0, value * this.typeItemHeight(type) + el.scrollHeight / 2 - 80 - 7);
          }
        }
      },

      scrollDown(step) {
        if (!this.currentScrollbar) {
          this.emitSelectRange('hours');
        }

        const label = this.currentScrollbar;
        const hoursList = this.hoursList;
        let now = this[label];

        if (this.currentScrollbar === 'hours') {
          let total = Math.abs(step);
          step = step > 0 ? 1 : -1;
          let length = hoursList.length;
          while (length-- && total) {
            now = (now + step + hoursList.length) % hoursList.length;
            if (hoursList[now]) {
              continue;
            }
            total--;
          }
          if (hoursList[now]) return;
        } else {
          now = (now + step + 60) % 60;
        }

        this.modifyDateField(label, now);
        this.adjustSpinner(label, now);
      },
      amPm(hour) {
        let shouldShowAmPm = this.amPmMode.toLowerCase() === 'a';
        if (!shouldShowAmPm) return '';
        let isCapital = this.amPmMode === 'A';
        let content = (hour < 12) ? ' am' : ' pm';
        if (isCapital) content = content.toUpperCase();
        return content;
      },
      typeItemHeight(type) {
        return this.$refs[type].$el.querySelector('li').offsetHeight;
      },
      scrollBarHeight(type) {
        return this.$refs[type].$el.offsetHeight;
      },
      scrollScrollHeight(type) {
        return this.$refs[type].wrap.querySelector('ul').offsetHeight;
      }
    }
  };
</script>

<style>
 /* .el-time-spinner .el-time-spinner__list{
    /deep/ .el-time-spinner__list {
      &:before, &:after {
        display: none;
      }
    }
  }*/
  .el-time-spinner .el-time-spinner__list::before, .el-time-spinner .el-time-spinner__list::after {
    display: none;
  }
</style>