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
        :readonly="inputReadOnly"
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
        :readonly="inputReadOnly"
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
              :class="dateDayItemClasse(item, index)"
              :index="index"
              @click="selectDate(item, index)"
              @mouseover="overDate(item, index)"
            >
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
      yearList: [],
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
    inputReadOnly: {
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
      // Options: 'both', 'from', 'to'
      type: String,
      default: 'both',
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
    allowSelectSameDay: {
      type: Boolean,
      default: true,
    }
  },
  beforeMount() {
    // set yearList
    for (let index = 0; index < 12; index++) { 
      this.yearList.push(new Date().getFullYear() + index)
    }
  },
  methods: {
    togglePanel() {
      if (this.disabled) {
        return false;
      }

      if(this.panelState === false) {
        // Is opening panel
        this.setUpdateCalendarOnDate()
      } else {
        // Is closing panel
      }
      this.panelState = !this.panelState;
      this.rangeStart = false;
    },
    setUpdateCalendarOnDate() {
      console.log('setUpdateCalendarOnDate')
      debugger
      let now = new Date();
      this.tmpMonth = typeof this.tmpStartMonth === "number" && !isNaN(this.tmpStartMonth) ? this.tmpStartMonth :  now.getMonth();
      this.tmpYear = typeof this.tmpStartYear === "number" && !isNaN(this.tmpStartYear) ? this.tmpStartYear :  now.getFullYear();
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
            new Date(this.tmpYear, month, item.value).getTime() > new Date(this.tmpStartYear, this.tmpStartMonth, this.tmpStartDate).getTime() &&
            new Date(this.tmpYear, month, item.value).getTime() < new Date(this.tmpEndYear, this.tmpEndMonth, this.tmpEndDate).getTime() 
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
          this.updateTmpValues([]);
          this.tmpEndYear = this.tmpStartYear = this.tmpYear;
          this.tmpEndMonth = this.tmpStartMonth = this.tmpMonth;
          this.tmpEndDate = this.tmpStartDate = date.value;
          this.tmpStartIndex = index;

          this.rangeStart = true;
        } else if (this.range && this.rangeStart) {
          if(this.allowSelectSameDay == false && date.value === this.tmpStartDate && this.tmpMonth === this.tmpStartMonth) {
            return false;
          }
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
      this.$emit("input", this.range ? [] : "");
    },
    closeIcon(color) {
      return `<g
        transform="matrix(0.1,0,0,-0.1,-30.5,1264.615)"
        fill="${color}">
        <path d="m 1395,12634 c -85,-19 -167,-51 -243,-95 -69,-41 -639,-599 -707,-694 -101,-141 -140,-263 -140,-440 0,-169 36,-293 125,-427 29,-43 705,-726 2149,-2170 L 4985,6400 2574,3988 C 1218,2630 450,1855 427,1819 339,1682 306,1570 306,1400 c -1,-181 37,-302 139,-445 68,-95 488,-503 557,-544 273,-159 604,-143 853,42 22,17 986,976 2143,2131 L 6400,4985 8803,2584 C 9959,1429 10923,470 10945,453 c 69,-51 130,-82 224,-113 208,-70 431,-44 629,71 69,41 489,449 557,544 101,141 140,263 140,440 0,166 -36,290 -121,422 -25,39 -746,767 -2148,2171 l -2411,2412 2407,2408 c 2207,2208 2162,2161 2219,2303 75,187 77,392 4,572 -53,132 226,-143 -315,400 -289,291 -252,248 -285,272 -141,101 -263,140 -440,140 -166,0 -289,-35 -420,-120 -41,-26 -724,-702 -2172,-2149 L 6550,7965 4138,10376 c -1454,1452 -2132,2123 -2173,2150 -64,41 -149,78 -230,101 -79,22 -258,26 -340,7 z" />
      </g>`
    },
    updateTmpValues(value){
      if (Array.isArray(value) && value.length === 0) {
        this.tmpStartYear = null;
        this.tmpStartMonth = null;
        this.tmpStartDate = null;

        this.tmpEndYear = null;
        this.tmpEndMonth = null;
        this.tmpEndDate = null;
        this.tmpEndIndex = null;
      } else if(value.length === 2 && value[0] !== null && value[1] !== null){
        let startDate = new Date(value[0])
        let endDate = new Date(value[1])
           
        this.tmpStartYear = startDate.getFullYear();
        this.tmpStartMonth = startDate.getMonth();
        this.tmpStartDate = startDate.getDate();

        this.tmpEndYear = endDate.getFullYear();
        this.tmpEndMonth = endDate.getMonth();
        this.tmpEndDate = endDate.getDate();
      }
    },
    dateDayItemClasse(item, index) {

      let isInterval = false;
      let isStart = false;
      let isEnd = false;

      // Range started
      if(this.rangeStart && this.tmpEndIndex) {
        // Same Year
        if(this.tmpStartYear == this.tmpYear) {

          // Same year and same month
          if(this.tmpMonth == this.tmpStartMonth){
            isInterval = index > this.tmpStartIndex && index < this.tmpEndIndex || index < this.tmpStartIndex && index > this.tmpEndIndex;
            isStart = index == this.tmpStartIndex && this.tmpEndIndex > index  && this.tmpEndIndex || index == this.tmpEndIndex && this.tmpStartIndex > index && this.tmpEndIndex;
            isEnd =   index == this.tmpEndIndex && this.tmpStartIndex < index  && this.tmpEndIndex || index == this.tmpStartIndex && this.tmpEndIndex < index && this.tmpEndIndex;
          } 
          // Month Ahead
          else if (this.tmpMonth > this.tmpStartMonth) {
            isInterval = index <= this.tmpEndIndex - 1;
            isStart = false;
            isEnd = index == this.tmpEndIndex;
          }
          // Month Back
          else if (this.tmpMonth < this.tmpStartMonth) {
            isInterval = index >= this.tmpEndIndex + 1;
            isStart = index == this.tmpEndIndex;
            isEnd = false;
          }

        }
        // year back
        else if (this.tmpStartYear > this.tmpYear) {
          isInterval = this.tmpYear && index >= this.tmpEndIndex + 1;
          isStart = index == this.tmpEndIndex;
          isEnd = false;
        }
        // year ahead
        else {
          isInterval = index <= this.tmpEndIndex - 1;
          isStart = false;
          isEnd = index == this.tmpEndIndex;
        }
      }

      return {
        preMonth: item.previousMonth,
        nextMonth: item.nextMonth,
        invalid: this.validateDate(item),
        firstItem: index % 7 === 0,
        interval: 
          this.range && this.isSelected('date', item) || isInterval,
        start:
          this.range && !this.rangeStart && this.isFirstDaySelected(item) || isStart,
        end:
          this.range && !this.rangeStart && this.isLastDaySelected(item) || isEnd,
      }
    }
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
      if(v === []){
        this.clear()
      }
      const newDate = new Date(v)
      this.date = newDate.getDate()
      this.updateTmpValues(v)
    }
  },
  computed: {
    dateList() {
      let currentMonthLength = new Date(
        this.tmpYear,
        this.tmpMonth + 1,
        0
      ).getDate();
      let dateList = [];
      for (let index = 0; index < currentMonthLength; index++) { 
        dateList.push({currentMonth: true, value: index + 1,})
      }
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

      if (this.showFromTo === 'both') {
        return this.range ? "From " + dateFrom + " to " + dateTo : this.value;
      } else if(this.showFromTo == 'from') {
        return this.range ? dateFrom : this.value;
      } else if(this.showFromTo == 'to') {
        return this.range ? dateTo : this.value;
      } else {
        return this.range ? dateFrom + " - " + dateTo : this.value;
      }
    },
  },
  filters: {
    week: (item, lang) => {
      switch (lang) {
        case "ar":
          return {
            0: "السبت",
            1: "الأحد",
            2: "الاثنين",
            3: "الثلاثاء",
            4: "الأربعاء",
            5: "الجمعة",
            6: "جمعة",
          }[item];
        case "bg":
          return {
            0: "нд",
            1: "пн",
            2: "вт",
            3: "ср",
            4: "чт",
            5: "пт",
            6: "сб",
          }[item];
        case "ca":
          return {
            0: "dg",
            1: "dl",
            2: "dt",
            3: "dc",
            4: "dj",
            5: "dv",
            6: "ds",
          }[item];
        case "cs":
          return {
            0: "Ne",
            1: "Po",
            2: "Út",
            3: "St",
            4: "Čt",
            5: "Pá",
            6: "So",
          }[item];
        case "da":
          return {
            0: "Søn.",
            1: "Man.",
            2: "Tirs.",
            3: "Ons.",
            4: "Tors.",
            5: "Fre.",
            6: "Lør.",
          }[item];
        case "de":
          return {
            0: "So",
            1: "Mo",
            2: "Di",
            3: "Mi",
            4: "Do",
            5: "Fr",
            6: "Sa",
          }[item];
        case "el":
          return {
            0: "Κυρ.",
            1: "Δευ.",
            2: "Τρ.",
            3: "Τετ.",
            4: "Πέμ.",
            5: "Παρ.",
            6: "Σάβ.",
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
        case "et":
          return {
            0: "P",
            1: "E",
            2: "T",
            3: "K",
            4: "N",
            5: "R",
            6: "L",
          }[item];
        case "eu":
          return {
            0: "ig.",
            1: "al.",
            2: "ar.",
            3: "az.",
            4: "og.",
            5: "or.",
            6: "lr.",
          }[item];
        case "fi":
          return {
            0: "Su",
            1: "Ma",
            2: "Ti",
            3: "Ke",
            4: "To",
            5: "Pe",
            6: "La",
          }[item];
        case "fr":
          return {
            0: "Di",
            1: "Lu",
            2: "Ma",
            3: "Me",
            4: "Je",
            5: "Ve",
            6: "Sa",
          }[item];
        case "he":
          return {
            0: "ראשון",
            1: "שני",
            2: "שלישי",
            3: "רביעי",
            4: "חמישי",
            5: "שישי",
            6: "שבת",
          }[item];
        case "hu":
          return {
            0: "Vas.",
            1: "H.",
            2: "K.",
            3: "Sze.",
            4: "Csüt.",
            5: "P.",
            6: "Szo.",
          }[item];
        case "id":
          return {
            0: "Min",
            1: "Sen",
            2: "Sel",
            3: "Rab",
            4: "Kam",
            5: "Jum",
            6: "Sab",
          }[item];
        case "it":
          return {
            0: "dom.",
            1: "lun.",
            2: "mar.",
            3: "mer.",
            4: "gio.",
            5: "ven.",
            6: "sab.",
          }[item];
        case "ja":
          return {
            0: "日曜",
            1: "月曜",
            2: "火曜",
            3: "水曜",
            4: "木曜",
            5: "金曜",
            6: "土曜",
          }[item];
        case "ko":
          return {
            0: "일요일",
            1: "월요일",
            2: "화요일",
            3: "수요일",
            4: "목요일",
            5: "금요일",
            6: "토요일",
          }[item];
        case "lv":
          return {
            0: "Sv.",
            1: "P.",
            2: "O.",
            3: "T.",
            4: "C.",
            5: "Pk.",
            6: "S.",
          }[item];
        case "ms":
          return {
            0: "Ahad",
            1: "isnin",
            2: "masanya",
            3: "Rabu",
            4: "Khamis",
            5: "Jumaat",
            6: "Sabtu",
          }[item];
        case "nl":
          return {
            0: "zon",
            1: "ma",
            2: "di",
            3: "woe",
            4: "don",
            5: "vrij",
            6: "zat",
          }[item];
        case "no":
          return {
            0: "Sø.",
            1: "Ma.",
            2: "Ti.",
            3: "On.",
            4: "To.",
            5: "Fr.",
            6: "Lø.",
          }[item];
        case "pl":
          return {
            0: "ndz.",
            1: "pn.",
            2: "wt.",
            3: "śr.",
            4: "czw.",
            5: "pt.",
            6: "sob.",
          }[item];
        case "pt":
          return {
            0: "Dom",
            1: "Seg",
            2: "Ter",
            3: "Qua",
            4: "Qui",
            5: "Sex",
            6: "Sab",
          }[item];
        case "ru":
          return {
            0: "ВСК",
            1: "ПНД",
            2: "ВТР",
            3: "СРД",
            4: "ЧТВ",
            5: "ПТН",
            6: "СБТ",
          }[item];
        case "sk":
          return {
            0: "Sø.",
            1: "Ma.",
            2: "Ti.",
            3: "On.",
            4: "To.",
            5: "Fr.",
            6: "Lø.",
          }[item];
        case "sr":
          return {
            0: "Нед.",
            1: "Пон.",
            2: "Ут.",
            3: "Ср.",
            4: "Чет.",
            5: "Пет.",
            6: "Суб.",
          }[item];
        case "sv":
          return {
            0: "Sön",
            1: "Mån",
            2: "Tis",
            3: "Ons",
            4: "Tors",
            5: "Fre",
            6: "Lör",
          }[item];
        case "th":
          return {
            0: "อา.",
            1: "จ.",
            2: "อ.",
            3: "พ.",
            4: "พฤ.",
            5: "ศ.",
            6: "ส.",
          }[item];
        case "tl":
          return {
            0: "Lun",
            1: "Mar",
            2: "Miy",
            3: "Huw",
            4: "Biy",
            5: "Sab",
            6: "Lin",
          }[item];
        case "tr":
          return {
            0: "Paz",
            1: "Pzt",
            2: "Sa",
            3: "Çrs",
            4: "Prs",
            5: "Cum",
            6: "Cmt",
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
        case "vi":
          return {
            0: "CN",
            1: "T2",
            2: "T3",
            3: "T4",
            4: "T5",
            5: "T6",
            6: "T7",
          }[item];
        case "zh":
          return {
            0: "日",
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
          }[item];
        case "zh-TW":
          return {
            0: "日",
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
          }[item];
        default:
          return {
            0: "Su",
            1: "Mo",
            2: "Tu",
            3: "We",
            4: "Th",
            5: "Fr",
            6: "Sa",
          }[item];
      }
    },
    month: (item, lang) => {
      switch (lang) {
        case "ar":
          return {
            1: "كانون الثاني",
            2: "شهر فبراير",
            3: "مارس",
            4: "أبريل",
            5: "قد",
            6: "يونيو",
            7: "تموز",
            8: "أغسطس",
            9: "شهر تسعة",
            10: "اكتوبر",
            11: "شهر نوفمبر",
            12: "ديسمبر",
          }[item];
        case "bg":
          return {
            1: "ян.",
            2: "февр.",
            3: "март",
            4: "април",
            5: "май",
            6: "юни",
            7: "юли",
            8: "авг.",
            9: "септ.",
            10: "окт.",
            11: "ноем.",
            12: "дек.",
          }[item];
        case "ca":
          return {
            1: "gen.",
            2: "febr.",
            3: "març",
            4: "abr.",
            5: "maig",
            6: "juny",
            7: "jul.",
            8: "ag.",
            9: "set.",
            10: "oct.",
            11: "nov.",
            12: "des.",
          }[item];
        case "cs":
          return {
            1: "ed.",
            2: "ún.",
            3: "břez.",
            4: "dub.",
            5: "květ.",
            6: "červ.",
            7: "červen.",
            8: "srp.",
            9: "září",
            10: "řij.",
            11: "list.",
            12: "pros.",
          }[item];
        case "da":
          return {
            1: "an.",
            2: "febr.",
            3: "marts",
            4: "april",
            5: "maj",
            6: "juni",
            7: "juli",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "de":
          return {
            1: "Jän.",
            2: "Feb.",
            3: "März",
            4: "Apr.",
            5: "Mai",
            6: "Juni",
            7: "Juli",
            8: "Aug.",
            9: "Sept.",
            10: "Okt.",
            11: "Nov.",
            12: "Dez.",
          }[item];
        case "el":
          return {
            1: "Ίαν.",
            2: "Φεβρ.",
            3: "Μάρτ",
            4: "Άπρ.",
            5: "Μάϊος",
            6: "Ίούν.",
            7: "Ίούλ.",
            8: "Αύγ.",
            9: "Σεπτ.",
            10: "Όκτ.",
            11: "Νοέµ.",
            12: "Δεκ.",
          }[item];
        case "es":
          return {
            1: "enero",
            2: "feb.",
            3: "marzo",
            4: "abr.",
            5: "mayo",
            6: "jun.",
            7: "jul.",
            8: "agosto",
            9: "sept.",
            10: "oct.",
            11: "nov.",
            12: "dic.",
          }[item];
        case "et":
          return {
            1: "jaan.",
            2: "veebr.",
            3: "märts",
            4: "apr.",
            5: "mai",
            6: "juuni",
            7: "juuli",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "dets.",
          }[item];
        case "eu":
          return {
            1: "urt.",
            2: "ots.",
            3: "mar.",
            4: "api.",
            5: "mai.",
            6: "eka.",
            7: "uzt.",
            8: "abu.",
            9: "ira.",
            10: "urr.",
            11: "aza.",
            12: "abe.",
          }[item];
        case "fi":
          return {
            1: "tammik.",
            2: "helmik.",
            3: "maalisk.",
            4: "huhtik.",
            5: "toukok.",
            6: "kesäk.",
            7: "heinäk.",
            8: "elok.",
            9: "syysk.",
            10: "lokak.",
            11: "marrask.",
            12: "jouluk.",
          }[item];
        case "fr":
          return {
            1: "janv.",
            2: "févr.",
            3: "mars",
            4: "avril",
            5: "mai",
            6: "juin",
            7: "juil.",
            8: "août",
            9: "sept.",
            10: "oct.",
            11: "nov.",
            12: "déc.",
          }[item];
        case "he":
          return {
            1: "Tishri",
            2: "Cheshvan",
            3: "Kislev",
            4: "Tevet",
            5: "Shevat",
            6: "Adar",
            7: "Nisan",
            8: "Iyar",
            9: "Sivan",
            10: "Tammuz",
            11: "Av",
            12: "Elul",
          }[item];
        case "hu":
          return {
            1: "jan.",
            2: "feb.",
            3: "márc.",
            4: "ápr.",
            5: "máj",
            6: "jún.",
            7: "júl.",
            8: "aug.",
            9: "szept.",
            10: "okt.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "id":
          return {
            1: "Djan.",
            2: "Peb.",
            3: "Mrt.",
            4: "Apr.",
            5: "Mai",
            6: "Djuni",
            7: "Djuli",
            8: "Ag.",
            9: "sept.",
            10: "okt.",
            11: "Nop.",
            12: "Des.",
          }[item];
        case "it":
          return {
            1: "genn.",
            2: "febbr.",
            3: "mar.",
            4: "apr.",
            5: "magg.",
            6: "giugno",
            7: "luglio",
            8: "ag.",
            9: "sett.",
            10: "ott.",
            11: "nov.",
            12: "dic.",
          }[item];
        case "ja":
          return {
            1: "一月",
            2: "二月",
            3: "三月",
            4: "四月",
            5: "五月",
            6: "六月",
            7: "七月",
            8: "八月",
            9: "九月",
            10: "十月",
            11: "十一月",
            12: "十二月",
          }[item];
        case "ko":
          return {
            1: "일월",
            2: "이월",
            3: "삼월",
            4: "사월",
            5: "오월",
            6: "유월",
            7: "칠월",
            8: "팔월",
            9: "구월",
            10: "시월",
            11: "십일월",
            12: "십이월",
          }[item];
        case "lv":
          return {
            1: "jan.",
            2: "feb.",
            3: "marts",
            4: "apr.",
            5: "maijs",
            6: "junijs",
            7: "julijs",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "ms":
          return {
            1: "Jan.",
            2: "Feb.",
            3: "Mac",
            4: "Apr.",
            5: "Mei",
            6: "Jun",
            7: "Julai",
            8: "Og.",
            9: "Sept.",
            10: "Okt.",
            11: "Nov.",
            12: "Dis.",
          }[item];
        case "nl":
          return {
            1: "jan.",
            2: "feb.",
            3: "maart",
            4: "apr.",
            5: "mei",
            6: "juni",
            7: "juli",
            8: "aug.",
            9: "sept.",
            10: "oct.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "no":
          return {
            1: "jan.",
            2: "febr.",
            3: "mars",
            4: "april",
            5: "mai",
            6: "juni",
            7: "juli",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "des.",
          }[item];
        case "pl":
          return {
            1: "stycz.",
            2: "luty",
            3: "mar.",
            4: "kwiec.",
            5: "maj",
            6: "czerw.",
            7: "lip.",
            8: "sierp.",
            9: "wrzes.",
            10: "paźdz.",
            11: "listop.",
            12: "grudz.",
          }[item];
        case "pt":
          return {
            1: "Jan",
            2: "Fev",
            3: "Mar",
            4: "Abr",
            5: "Mai",
            6: "Jun",
            7: "Jul",
            8: "Ago",
            9: "Set",
            10: "Out",
            11: "Nov",
            12: "Dez",
          }[item];
        case "ru":
          return {
            1: "янв.",
            2: "февр.",
            3: "март",
            4: "апр.",
            5: "май",
            6: "июнь",
            7: "июль",
            8: "авг.",
            9: "сент.",
            10: "окт.",
            11: "ноябрь",
            12: "дек.",
          }[item];
        case "sk":
          return {
            1: "l'ad.",
            2: "ún.",
            3: "brez.",
            4: "dub.",
            5: "kvet.",
            6: "červ.",
            7: "červen.",
            8: "srp.",
            9: "zári.",
            10: "ruj.",
            11: "list.",
            12: "pros.",
          }[item];
        case "sr":
          return {
            1: "jan.",
            2: "febr.",
            3: "mart",
            4: "april",
            5: "maj",
            6: "juni",
            7: "juli",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "sv":
          return {
            1: "jan.",
            2: "febr.",
            3: "mars",
            4: "april",
            5: "maj",
            6: "juni",
            7: "juli",
            8: "aug.",
            9: "sept.",
            10: "okt.",
            11: "nov.",
            12: "dec.",
          }[item];
        case "th":
          return {
            1: "มกราคม",
            2: "กุมภาพันธ์",
            3: "มีนาคม",
            4: "เมษายน",
            5: "พฤษภาคม",
            6: "มิถุนายน",
            7: "กรกฎาคม",
            8: "สิงหาคม",
            9: "กันยายน",
            10: "ตุลาคม",
            11: "พฤศจิกายน",
            12: "ธันวาคม",
          }[item];
        case "tl":
          return {
            1: "Ene",
            2: "Peb",
            3: "Mar",
            4: "Abr",
            5: "May",
            6: "Hun",
            7: "Hul",
            8: "Ago",
            9: "Set",
            10: "Okt",
            11: "Nob",
            12: "Dis",
          }[item];
        case "tr":
          return {
            1: "ocak",
            2: "şubat",
            3: "mart",
            4: "nisan",
            5: "mayıs",
            6: "haziran",
            7: "temmuz",
            8: "ağustos",
            9: "eylül",
            10: "ekim",
            11: "kasım",
            12: "aralık",
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
        case "vi":
          return {
            1: "Một",
            2: "Hai",
            3: "Ba",
            4: "Tư",
            5: "Năm",
            6: "Sáu",
            7: "Bảy",
            8: "Tám",
            9: "Chín",
            10: "Mười",
            11: "Mười Một",
            12: "Mười Hai",
          }[item];
        case "zh":
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
        case "zh-TW":
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
        default:
          // defaults to english
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
.date-list li.interval {
  background: var(--header-color);
  opacity: 0.7;
  color: #fff;
}
.date-list li.start, .date-list li.end {
  color: #FFF;
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
