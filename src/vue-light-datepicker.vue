<template>
  <div
    class="date-picker" 
    :style="{
      '--header-color':headerColor,
      '--header-tex-color':headerTexColor,
      '--selected-cay-color':selectedDayColor,
      '--clean-icon-color':cleanIconColor,
    }"
  >
    <div
      v-if="popup"
      v-show="panelState"
      class="date-picker-modal-bg"
      @click="forceClose()"
    ></div>
    <div
      class="date-picker-wrapper"
      :class="{ disabled: disabled }"
      @mouseenter="showCancel = true"
      @mouseleave="showCancel = false"
    >
      <!-- <div v-if="isValueEmpty" class="input" @click="togglePanel">Select {{range? 'dates':'date'}}</div> -->
      <input
        type="text"
        inputmode='none'
        v-if="isValueEmpty"
        :style="inputStyle"
        class="input"
        @click="togglePanel"
        :placeholder="`Select date${range ? 's' : ''}`"
      />
      <!-- <div v-else class="input" @click="togglePanel" v-text="pickerOutput"></div> -->
      <input
        type="text"
        inputmode='none'
        v-else
        :style="inputStyle"
        class="input"
        @click="togglePanel"
        :placeholder="pickerOutput"
        :value="pickerOutput"
      />
      <transition name="fade">
        <!-- <img
          class="cancel-btn"
          :class="{ outside: clearOutside }"
          src="./close.svg"
          v-if="enableClear"
          v-show="showCancel"
          @click="clear"
        /> -->
        <svg
          version="1.0"
          xmlns="http://www.w3.org/2000/svg"
          width="1280.000000pt"
          height="1280.000000pt"
          viewBox="0 0 1280.000000 1280.000000"
          preserveAspectRatio="xMidYMid meet"
          class="cancel-btn"
          :class="{ outside: clearOutside }"
          v-if="enableClear"
          v-show="showCancel"
          @click="clear"
          v-html="closeIcon(cleanIconColor)"
        >
        </svg>
      </transition>
    </div>
    <transition name="toggle">
      <div class="date-panel" v-show="panelState" :style="coordinates" :class="{'rounded':roundCorners}">
        <div v-if="showClosePicker" class="closePicker" :style="{'background':headerColor, 'color':headerTexColor}">
          <svg
            version="1.0"
            xmlns="http://www.w3.org/2000/svg"
            width="1280.000000pt"
            height="1280.000000pt"
            viewBox="0 0 1280.000000 1280.000000"
            preserveAspectRatio="xMidYMid meet"
            @click="forceClose()"
            v-html="closeIcon('#FFF')"
          >
          </svg>
        </div>
        <div class="panel-header" :style="{'background':headerColor, 'color':headerTexColor}" v-show="panelType !== 'year'">
          <div class="arrow-left" @click="prevMonthPreview()">&lt;</div>
          <div class="year-month-box">
            <div
              class="year-box"
              @click="chType('year')"
              v-text="tmpYear"
            ></div>
            <div class="month-box" @click="chType('month')">
              {{ (tmpMonth + 1) | month(language) }}
            </div>
          </div>
          <div class="arrow-right" @click="nextMonthPreview()">&gt;</div>
        </div>
        <div class="panel-header" :style="{'background':headerColor, 'color':headerTexColor}" v-show="panelType === 'year'">
          <div class="arrow-left" @click="chYearRange(0)">&lt;</div>
          <div class="year-range">
            <span v-text="yearList[0]"></span> -
            <span v-text="yearList[yearList.length - 1]"></span>
          </div>
          <div class="arrow-right" @click="chYearRange(1)">&gt;</div>
        </div>
        <div class="type-year" v-show="panelType === 'year'">
          <ul class="year-list">
            <li
              v-for="(item, index) in yearList"
              :key="index"
              v-text="item"
              :class="{
                selected: isSelected('year', item),
                invalid: validateYear(item),
              }"
              :style="{
                  'background-color':isSelected('year', item) ? headerColor: 'inherit',
                  '--color-hover':headerColor
                }"
              @click="selectYear(item)"
            ></li>
          </ul>
        </div>
        <div class="type-month" v-show="panelType === 'month'">
          <ul class="month-list">
            <li
              v-for="(item, index) in monthList"
              :key="index"
              :class="{
                selected: isSelected('month', index),
                invalid: validateMonth(index),
              }"
              :style="{
                  'background-color':isSelected('month', index) ? headerColor: 'inherit',
                  '--color-hover':headerColor
                }"
              @click="selectMonth(index)"
            >
              {{ item | month(language) }}
            </li>
          </ul>
        </div>
        <div class="type-date" v-show="panelType === 'date'">
          <ul class="weeks" :style="{'background':headerColor, color:headerTexColor}">
            <li v-for="(item, index) in weekList" :key="index">{{ item | week(language) }}</li>
          </ul>
          <ul class="date-list">
            <li
              v-for="(item, index) in dateList"
              :key="index"
              :class="{
                preMonth: item.previousMonth,
                nextMonth: item.nextMonth,
                invalid: validateDate(item),
                firstItem: index % 7 === 0,
                interval: rangeStart && index > tmpStartIndex && index < tmpEndIndex || range && isSelected('date', item),
                start: rangeStart && index == tmpStartIndex && tmpEndIndex > index || range && !rangeStart && isFirstDaySelected(item),
                end: rangeStart && index == tmpEndIndex && tmpStartIndex < index || range && !rangeStart && isLastDaySelected(item)
              }"
              :index="index"
              @click="selectDate(item, index)"
              @mouseover="overDate(item, index)"
            >
                <!-- start: rangeStart && index == tmpStartIndex && tmpEndIndex > index,
                end: rangeStart && index == tmpEndIndex && tmpStartIndex < index -->
              <div
                class="message"
                :class="{ selected: isSelected('date', item) }"
              >
                <div class="bg" :style="{'background-color':selectedDayColor}"></div>
                <span v-text="item.value"></span>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default /*#__PURE__*/ {
  name: "VueLightDatepicker", // vue component name
  data() {
    let now = new Date();
    return {
      showCancel: false,
      panelState: false,
      panelType: "date",
      coordinates: {},
      year: now.getFullYear(),
      month: now.getMonth(),
      date: null,
      tmpYear: now.getFullYear(),
      tmpMonth: now.getMonth(),
      tmpStartYear: null,
      tmpStartMonth: null,
      tmpStartDate: null,
      tmpStartIndex: null,
      tmpEndYear: null,
      tmpEndMonth: null,
      tmpEndDate: null,
      tmpEndIndex: null,
      minYear: Number,
      minMonth: Number,
      minDate: Number,
      maxYear: Number,
      maxMonth: Number,
      maxDate: Number,
      yearList: Array.from(
        { length: 12 },
        (value, index) => new Date().getFullYear() + index
      ),
      monthList: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
      weekList: [0, 1, 2, 3, 4, 5, 6],
      rangeStart: false,
    };
  },
  props: {
    language: { default: "en" },
    min: { default: "1970-01-01" },
    max: { default: "3016-01-01" },
    value: {
      type: [String, Array],
      default: "",
    },
    popup: {
      type: Boolean,
      default: false,
    },
    range: {
      type: Boolean,
      default: false,
    },
    enableClear: {
      type: Boolean,
      default: true,
    },
    clearOutside: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    showFromTo: {
      type: Boolean,
      default: true,
    },
    showClosePicker: {
      type: Boolean,
      default: true,
    },
    roundCorners: {
      type: Boolean,
      default: false,
    },
    headerColor: {
      type: String,
      default: '#0097a7'
    },
    headerTexColor: {
      type: String,
      default: '#FFF'
    },
    selectedDayColor: {
      type: String,
      default: '#0097a7'
    },
    inputStyle: {
      type: [Object, String],
      default() {
        return {}
      },
      required: false
    },
    cleanIconColor: {
      type: String,
      default: '#000000'
    },
  },
  methods: {
    togglePanel() {
      if (this.disabled) {
        return false;
      }
      this.panelState = !this.panelState;
      this.rangeStart = false;
    },
    isSelected(type, item) {
      switch (type) {
        case "year":
          if (!this.range) return item === this.tmpYear;
          return (
            new Date(item, 0).getTime() >=
              new Date(this.tmpStartYear, 0).getTime() &&
            new Date(item, 0).getTime() <=
              new Date(this.tmpEndYear, 0).getTime()
          );
        case "month":
          if (!this.range)
            return item === this.tmpMonth && this.year === this.tmpYear;
          return (
            new Date(this.tmpYear, item).getTime() >=
              new Date(this.tmpStartYear, this.tmpStartMonth).getTime() &&
            new Date(this.tmpYear, item).getTime() <=
              new Date(this.tmpEndYear, this.tmpEndMonth).getTime()
          );
        case "date":
          if (!this.range)
            return (
              this.date === item.value &&
              this.month === this.tmpMonth &&
              item.currentMonth
            );
          let month = this.tmpMonth;
          item.previousMonth && month--;
          item.nextMonth && month++;
          return (
            new Date(this.tmpYear, month, item.value).getTime() >
              new Date(this.tmpStartYear, this.tmpStartMonth, this.tmpStartDate).getTime() &&
            new Date(this.tmpYear, month, item.value).getTime() <
              new Date(this.tmpEndYear, this.tmpEndMonth, this.tmpEndDate).getTime()
          );
      }
    },
    isFirstDaySelected(item) {
      let month = this.tmpMonth;
      if(item.previousMonth){
        month--
      } else if (item.nextMonth) {
        month++
      }
      return new Date(this.tmpYear, month, item.value).getTime() ==
        new Date(this.tmpStartYear, this.tmpStartMonth, this.tmpStartDate).getTime()
    },
    isLastDaySelected(item) {
      let month = this.tmpMonth;
      if(item.previousMonth){
        month--
      } else if (item.nextMonth) {
        month++
      }
      return new Date(this.tmpYear, month, item.value).getTime() ==
        new Date(this.tmpEndYear, this.tmpEndMonth, this.tmpEndDate).getTime()
    },
    chType(type) {
      this.panelType = type;
    },
    chYearRange(next) {
      if (next) {
        this.yearList = this.yearList.map((i) => i + 12);
      } else {
        this.yearList = this.yearList.map((i) => i - 12);
      }
    },
    prevMonthPreview() {
      if (this.tmpMonth === 0) {
        this.tmpYear--;
        this.tmpMonth = 11;
      } else {
        this.tmpMonth--;
      }
    },
    nextMonthPreview() {
      if (this.tmpMonth === 11) {
        this.tmpYear++;
        this.tmpMonth = 0;
      } else {
        this.tmpMonth++;
      }
    },
    selectYear(year) {
      if (this.validateYear(year)) return;
      this.tmpYear = year;
      this.panelType = "month";
    },
    selectMonth(month) {
      if (this.validateMonth(month)) return;
      this.tmpMonth = month;
      this.panelType = "date";
    },
    selectDate(date, index) {
      setTimeout(() => {
        if (this.validateDate(date)) return;
        if (date.previousMonth) {
          if (this.tmpMonth === 0) {
            this.year -= 1;
            this.tmpYear -= 1;
            this.month = this.tmpMonth = 11;
          } else {
            this.month = this.tmpMonth - 1;
            this.tmpMonth -= 1;
          }
        } else if (date.nextMonth) {
          if (this.tmpMonth === 11) {
            this.year += 1;
            this.tmpYear += 1;
            this.month = this.tmpMonth = 0;
          } else {
            this.month = this.tmpMonth + 1;
            this.tmpMonth += 1;
          }
        }
        if (!this.range) {
          this.year = this.tmpYear;
          this.month = this.tmpMonth;
          this.date = date.value;
          let value = `${this.tmpYear}-${("0" + (this.month + 1)).slice(-2)}-${(
            "0" + this.date
          ).slice(-2)}`;
          this.$emit("input", value);
          this.panelState = false;
        } else if (this.range && !this.rangeStart) {
          this.tmpEndYear = this.tmpStartYear = this.tmpYear;
          this.tmpEndMonth = this.tmpStartMonth = this.tmpMonth;
          this.tmpEndDate = this.tmpStartDate = date.value;
          this.tmpStartIndex = index;

          this.rangeStart = true;
        } else if (this.range && this.rangeStart) {
          this.tmpEndYear = this.tmpYear;
          this.tmpEndMonth = this.tmpMonth;
          this.tmpEndDate = date.value;

          let d1 = new Date(
              this.tmpStartYear,
              this.tmpStartMonth,
              this.tmpStartDate
            ).getTime(),
            d2 = new Date(
              this.tmpEndYear,
              this.tmpEndMonth,
              this.tmpEndDate
            ).getTime();
          if (d1 > d2) {
            let tmpY, tmpM, tmpD;
            tmpY = this.tmpEndYear;
            tmpM = this.tmpEndMonth;
            tmpD = this.tmpEndDate;

            this.tmpEndYear = this.tmpStartYear;
            this.tmpEndMonth = this.tmpStartMonth;
            this.tmpEndDate = this.tmpStartDate;

            this.tmpStartYear = tmpY;
            this.tmpStartMonth = tmpM;
            this.tmpStartDate = tmpD;
          }
          let RangeStart = `${this.tmpStartYear}-${(
            "0" +
            (this.tmpStartMonth + 1)
          ).slice(-2)}-${("0" + this.tmpStartDate).slice(-2)}`;
          let RangeEnd = `${this.tmpEndYear}-${(
            "0" +
            (this.tmpEndMonth + 1)
          ).slice(-2)}-${("0" + this.tmpEndDate).slice(-2)}`;

          let value = [RangeStart, RangeEnd];
          this.$emit("input", value);

          this.rangeStart = false;
          this.panelState = false;
        }
      }, 0);
    },
    overDate(date, index){
      if(this.rangeStart){
        this.tmpEndIndex = index
      }
    },
    validateYear(year) {
      return year > this.maxYear || year < this.minYear ? true : false;
    },
    validateMonth(month) {
      if (
        new Date(this.tmpYear, month).getTime() >=
          new Date(this.minYear, this.minMonth - 1).getTime() &&
        new Date(this.tmpYear, month).getTime() <=
          new Date(this.maxYear, this.maxMonth - 1).getTime()
      ) {
        return false;
      }
      return true;
    },
    validateDate(date) {
      let mon = this.tmpMonth;
      if (date.previousMonth) {
        mon -= 1;
      } else if (date.nextMonth) {
        mon += 1;
      }
      if (
        new Date(this.tmpYear, mon, date.value).getTime() >=
          new Date(this.minYear, this.minMonth - 1, this.minDate).getTime() &&
        new Date(this.tmpYear, mon, date.value).getTime() <=
          new Date(this.maxYear, this.maxMonth - 1, this.maxDate).getTime()
      ) {
        return false;
      }
      return true;
    },
    close(e) {
      if (!this.$el.contains(e.target)) {
        this.panelState = false;
        this.rangeStart = false;
      }
    },
    forceClose() {
      this.panelState = false;
      this.rangeStart = false;
    },
    clear() {
      this.$emit("input", this.range ? ["", ""] : "");
    },
    closeIcon(color) {
      return `<g
        transform="matrix(0.1,0,0,-0.1,-30.5,1264.615)"
        fill="${color}">
        <path d="m 1395,12634 c -85,-19 -167,-51 -243,-95 -69,-41 -639,-599 -707,-694 -101,-141 -140,-263 -140,-440 0,-169 36,-293 125,-427 29,-43 705,-726 2149,-2170 L 4985,6400 2574,3988 C 1218,2630 450,1855 427,1819 339,1682 306,1570 306,1400 c -1,-181 37,-302 139,-445 68,-95 488,-503 557,-544 273,-159 604,-143 853,42 22,17 986,976 2143,2131 L 6400,4985 8803,2584 C 9959,1429 10923,470 10945,453 c 69,-51 130,-82 224,-113 208,-70 431,-44 629,71 69,41 489,449 557,544 101,141 140,263 140,440 0,166 -36,290 -121,422 -25,39 -746,767 -2148,2171 l -2411,2412 2407,2408 c 2207,2208 2162,2161 2219,2303 75,187 77,392 4,572 -53,132 226,-143 -315,400 -289,291 -252,248 -285,272 -141,101 -263,140 -440,140 -166,0 -289,-35 -420,-120 -41,-26 -724,-702 -2172,-2149 L 6550,7965 4138,10376 c -1454,1452 -2132,2123 -2173,2150 -64,41 -149,78 -230,101 -79,22 -258,26 -340,7 z" />
      </g>`
    },
  },
  watch: {
    min(v) {
      let minArr = v.split("-");
      this.minDate = Number(minArr[0]);
      this.minMonth = Number(minArr[1]);
      this.minYear = Number(minArr[2]);
    },
    max(v) {
      let maxArr = v.split("-");
      this.maxDate = Number(maxArr[0]);
      this.maxMonth = Number(maxArr[1]);
      this.maxYear = Number(maxArr[2]);
    },
    range(newVal, oldVal) {
      if (newVal === oldVal) return;
      if (
        newVal &&
        Object.prototype.toString.call(this.value).slice(8, -1) === "String"
      ) {
        this.$emit("input", ["", ""]);
      }
      if (
        !newVal &&
        Object.prototype.toString.call(this.value).slice(8, -1) === "Array"
      ) {
        this.$emit("input", "");
      }
    },
    value(v) {
      const newDate = new Date(v)
      this.date = newDate.getDate()

    }
  },
  computed: {
    dateList() {
      let currentMonthLength = new Date(
        this.tmpYear,
        this.tmpMonth + 1,
        0
      ).getDate();
      let dateList = Array.from(
        { length: currentMonthLength },
        (val, index) => {
          return {
            currentMonth: true,
            value: index + 1,
          };
        }
      );
      let startDay = new Date(this.tmpYear, this.tmpMonth, 1).getDay();
      let previousMongthLength = new Date(
        this.tmpYear,
        this.tmpMonth,
        0
      ).getDate();

      for (let i = 0, len = startDay; i < len; i++) {
        dateList = [
          { previousMonth: true, value: previousMongthLength - i },
        ].concat(dateList);
      }
      for (let i = dateList.length, item = 1; i < 42; i++, item++) {
        dateList[dateList.length] = { nextMonth: true, value: item };
      }
      return dateList;
    },
    isValueEmpty: function () {
      return !this.value[0] && !this.value[1];
    },
    pickerOutput: function () {
      if (this.range && this.value[0] && this.value[1]) {
        var dateFrom;
        var tempDateFrom;
        var dateTo;
        var tempDateTo;

        tempDateFrom = this.value[0].split("-");
        if (tempDateFrom && tempDateFrom.length >= 3) {
          dateFrom =
            tempDateFrom[2] + "/" + tempDateFrom[1] + "/" + tempDateFrom[0];
        } else {
          dateFrom = this.value[0];
        }

        tempDateTo = this.value[1].split("-");
        if (tempDateTo && tempDateTo.length >= 3) {
          dateTo = tempDateTo[2] + "/" + tempDateTo[1] + "/" + tempDateTo[0];
        } else {
          dateTo = this.value[1];
        }
      }

      if (this.showFromTo) {
        return this.range ? "From " + dateFrom + " to " + dateTo : this.value;
      } else {
        return this.range ? dateFrom + " - " + dateTo : this.value;
      }
    },
  },
  filters: {
    week: (item, lang) => {
      switch (lang) {
        case "en":
          return {
            0: "Su",
            1: "Mo",
            2: "Tu",
            3: "We",
            4: "Th",
            5: "Fr",
            6: "Sa",
          }[item];
        case "ch":
          return {
            0: "日",
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
          }[item];
        case "uk":
          return {
            0: "Пн",
            1: "Вт",
            2: "Ср",
            3: "Чт",
            4: "Пт",
            5: "Сб",
            6: "Нд",
          }[item];
        case "es":
          return {
            0: "Do",
            1: "Lu",
            2: "Ma",
            3: "Mi",
            4: "Ju",
            5: "Vi",
            6: "Sa",
          }[item];
        default:
          return item;
      }
    },
    month: (item, lang) => {
      switch (lang) {
        case "en":
          return {
            1: "Jan",
            2: "Feb",
            3: "Mar",
            4: "Apr",
            5: "May",
            6: "Jun",
            7: "Jul",
            8: "Aug",
            9: "Sep",
            10: "Oct",
            11: "Nov",
            12: "Dec",
          }[item];
        case "ch":
          return {
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
            7: "七",
            8: "八",
            9: "九",
            10: "十",
            11: "十一",
            12: "十二",
          }[item];
        case "uk":
          return {
            1: "Січень",
            2: "Лютий",
            3: "Березень",
            4: "Квітень",
            5: "Травень",
            6: "Червень",
            7: "Липень",
            8: "Серпень",
            9: "Вересень",
            10: "Жовтень",
            11: "Листопад",
            12: "Грудень",
          }[item];
        case "en":
          return {
            1: "Ene",
            2: "Feb",
            3: "Mar",
            4: "Abr",
            5: "May",
            6: "Jun",
            7: "Jul",
            8: "Ago",
            9: "Sep",
            10: "Oct",
            11: "Nov",
            12: "Dic",
          }[item];
        default:
          return item;
      }
    },
  },
  mounted() {
    this.$nextTick(() => {
      if (
        this.$el.parentNode.offsetWidth +
          this.$el.parentNode.offsetLeft -
          this.$el.offsetLeft <=
        300
      ) {
        this.coordinates = {
          right: "0",
          top: `${
            window.getComputedStyle(this.$el.children[0]).offsetHeight + 4
          }px`,
        };
      } else {
        this.coordinates = {
          left: "0",
          top: `${
            window.getComputedStyle(this.$el.children[0]).offsetHeight + 4
          }px`,
        };
      }
      let minArr = this.min.split("-");
      this.minYear = Number(minArr[0]);
      this.minMonth = Number(minArr[1]);
      this.minDate = Number(minArr[2]);

      let maxArr = this.max.split("-");
      this.maxYear = Number(maxArr[0]);
      this.maxMonth = Number(maxArr[1]);
      this.maxDate = Number(maxArr[2]);

      if (this.range) {
        if (
          Object.prototype.toString.call(this.value).slice(8, -1) !== "Array"
        ) {
          throw new Error("Binding value must be an array in range mode.");
        }
        if (this.value.length) {
          let rangeStart = this.value[0].split("-");
          let rangeEnd = this.value[1].split("-");
          this.tmpStartYear = Number(rangeStart[0]);
          this.tmpStartMonth = Number(rangeStart[1]) - 1;
          this.tmpStartDate = Number(rangeStart[2]);

          this.tmpEndYear = Number(rangeEnd[0]);
          this.tmpEndMonth = Number(rangeEnd[1]) - 1;
          this.tmpEndDate = Number(rangeEnd[2]);
        } else {
          this.$emit("input", ["", ""]);
        }
      }
      if (!this.value) {
        this.$emit("input", "");
      }
      window.addEventListener("click", this.close);
    });
  },
  beforeDestroy() {
    window.removeEventListener("click", this.close);
  },
};
</script>

<style scoped>
ul {
  padding: 0;
  margin: 0;
  list-style: none;
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  /* grid-template-columns: repeat(7, 14.285714286%); */
  grid-gap: 0;
}
.date-picker {
  position: relative;
  /* height: 32px; */
}

.date-picker .date-picker-modal-bg{
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1;
}

.date-picker-wrapper{
  position: relative;
  width: fit-content;
}

.disabled .date-picker-wrapper {
  color: #8ca0b3 !important;
  cursor: not-allowed;
}
.cancel-btn {
  height: 100%;
  width: 14px;
  right: 10px;
  top: 0;
  position: absolute;
  cursor: pointer;
}
.cancel-btn.outside {
  position: absolute;
  right: 9px;
}
.date-panel {
  position: absolute;
  z-index: 5000;
  box-sizing: border-box;
  width: 320px;
  background-color: #fff;
  transform: translateY(4px);
}
.date-panel.rounded {
  border-radius: 10px;
  overflow: hidden;
  padding-bottom: 2px;
}
.closePicker{
  text-align: right;
  padding: 2px 4px 0 0;
}
.closePicker svg{
  height: 15px;
  width: 15px;
  margin: 5px;
  cursor: pointer;
}
.panel-header {
  display: flex;
  flex-flow: row nowrap;
  width: 100%;
}
.arrow-left, .arrow-right {
  flex: 1;
  font-size: 20px;
  line-height: 2;
  text-align: center;
  cursor: pointer;
}
.year-range {
  font-size: 20px;
  line-height: 2;
  flex: 3;
  display: flex;
  justify-content: center;
}
.year-month-box {
  flex: 3;
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-around;
}
.type-year, .type-month, .date-list {
  background-color: #fff;
}
.year-box, .month-box {
  transition: all ease .1s;
  font-family: Roboto, sans-serif;
  flex: 1;
  text-align: center;
  font-size: 20px;
  line-height: 2;
  cursor: pointer;
}
.year-list, .month-list {
  display: flex;
  flex-flow: row wrap;
  justify-content: space-between;
}
.year-list li, .month-list li {
  font-family: Roboto, sans-serif;
  transition: all .45s cubic-bezier(0.23,1,0.32,1) 0ms;
  cursor: pointer;
  text-align: center;
  font-size: 20px;
  width: 33%;
  padding: 10px 0;
}
.year-list li:hover, .month-list li:hover {
  color: var(--color-hover);
}
.year-list li.selected, .month-list li.selected {
  color: #fff;
}
.year-list li.invalid, .month-list li.invalid {
  cursor: not-allowed;
  color: #ccc;
}
.date-list {
  flex-flow: row wrap;
  justify-content: space-between;
  /* padding: 0 5px 5px; */
}
.date-list .valid:hover {
  background-color: #eee;
}
.date-list li {
  /* transition: all ease .1s; */
  cursor: pointer;
  box-sizing: border-box;
  /* border-bottom: 1px solid #fff; */
  position: relative;
  aspect-ratio: 1;
}
.date-list li.start .message .bg {
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
  transform: scale(1) !important;
  opacity: 1 !important;
}
.date-list li.end .message .bg {
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
  transform: scale(1) !important;
  opacity: 1 !important;
}
.date-list li.interval .bg {
  /* background: var(--header-color);
  opacity: 0.8 !important;
  transform: scale(1) !important;
  color: #fff; */
}
.date-list li.interval {
  background: var(--header-color);
  opacity: 0.8;
  color: #fff;
}
.date-list li.start, .date-list li.end {
  color: #FFF;
}
.date-list li:not(.firstItem) {
  /* margin-left: 10px; */
}
.date-list li .message {
  font-family: Roboto, sans-serif;
  font-weight: 300;
  font-size: 14px;
  position: relative;
  height: 100%;
}
.date-list li .message.selected .bg {
  background-color: #0097a7;
  border-radius: 50%;
}
.date-list li .message.selected span {
  color: #fff;
}
.date-list li .message:not(.selected) .bg {
  transform: scale(0);
  opacity: 0;
}
.date-list li .message:not(.selected):hover .bg {
  background-color: #a74300;
  transform: scale(1);
  opacity: 1;
}
.date-list li .message:not(.selected):hover span {
  color: #fff;
}
.date-list li .message .bg {
  height: 100%;
  width: 100%;
  /* transition: all 450ms cubic-bezier(0.23,1,0.32,1) 0ms; */
  border-radius: 50%;
  position: absolute;
  z-index: 10;
  top: 0;
  left: 0;
}
.date-list li .message span {
  position: absolute;
  z-index: 20;
  left: 50%;
  top: 50%;
  transform: translate3d(-50%,-50%,0);
}
.date-list li.invalid {
  cursor: not-allowed;
  color: #ccc;
}
.weeks {
  justify-content: space-between;
}
.weeks li {
  font-weight: 600;
}
.weeks, .date-list {
  width: 100%;
  text-align: center;
}
.weeks .preMonth, .weeks .nextMonth, .date-list .preMonth, .date-list .nextMonth {
  color: #ccc;
}
.weeks li, .date-list li {
  font-family: Roboto, sans-serif;
  text-align: center;
  line-height: 30px;
}
.toggle-enter, .toggle-leave-active {
  opacity: 0;
  transform: translateY(-10px);
}
.toggle-enter-active, .toggle-leave-active {
  transition: all ease .2s;
}
.fade-enter, .fade-leave-active {
  opacity: 0;
  transform: scale3d(0,0,0);
}
.fade-enter-active, .fade-leave-active {
  transition: all ease .1s;
}
</style>
