<template>
  <div class="vdatetime-popup">
    <div class="vdatetime-popup__body">
      <table class="hrMinTable" v-if="!use12Hour">
            <tbody>
              <tr>
              <td class="td1">
                <button class="caretUpDown icon-angle-up" @click="hourUp()"></button>
                <div class="hrMinTxt">{{newDatetime.hour}}</div>
                <button class="caretUpDown icon-angle-down" @click="hourDn()"></button>
              </td>
              <td class="td2">
                <button class="caretUpDown icon-angle-up" @click="minUp()"></button>
                <div class="hrMinTxt">{{minute < 10 ? '0' + minute : minute}}</div>
                <button class="caretUpDown icon-angle-down" @click="minDn()"></button>
              </td>

              </tr>
              </tbody>
            </table>
            <table class="hrMinTable" v-else-if="use12Hour">
            <tbody>
              <tr>
              <td class="td1">
                <button class="caretUpDown icon-angle-up" @click="hourUp()"></button>
                <div class="hrMinTxt">{{nonMilitaryHr}}</div>
                <button class="caretUpDown icon-angle-down" @click="hourDn()"></button>
              </td>
              <td class="td2">
                <button class="caretUpDown icon-angle-up" @click="minUp()"></button>
                <div class="hrMinTxt">{{minute < 10 ? '0' + minute : minute}}</div>
                <button class="caretUpDown icon-angle-down" @click="minDn()"></button>
              </td>
              <td>
                <button class="ampm buttonHover" @click="toggleAmPm()">{{pm ? 'PM': 'AM'}}</button>
              </td>
              </tr>
              </tbody>
            </table>
    </div>
    <div class="vdatetime-popup__actions">
      <div class="vdatetime-popup__actions__button buttonHover" @click="now">
        <slot v-bind:step="step">Now</slot>
      </div>
      <div class="vdatetime-popup__actions__button vdatetime-popup__actions__button--cancel buttonHover" @click="cancel">
        <slot name="button-cancel__internal" v-bind:step="step">{{ phrases.cancel }}</slot>
      </div>
      <div class="vdatetime-popup__actions__button vdatetime-popup__actions__button--confirm buttonHover" @click="confirm">
        <slot name="button-confirm__internal" v-bind:step="step">{{ phrases.ok }}</slot>
      </div>
    </div>
  </div>
</template>

<script>
import { DateTime } from 'luxon'
import { createFlowManager, createFlowManagerFromType } from './util'
import DatetimeCalendar from './DatetimeCalendar'
import DatetimeYearPicker from './DatetimeYearPicker'
import DatetimeMonthPicker from './DatetimeMonthPicker'

const KEY_TAB = 9
const KEY_ENTER = 13
const KEY_ESC = 27

export default {
  components: {
    DatetimeCalendar,
    DatetimeYearPicker
  },

  props: {
    datetime: {
      type: DateTime,
      required: true
    },
    phrases: {
      type: Object,
      default () {
        return {
          cancel: 'Cancel',
          ok: 'Ok'
        }
      }
    },
    type: {
      type: String,
      default: 'date'
    },
    use12Hour: {
      type: Boolean,
      default: false
    },
    hourStep: {
      type: Number,
      default: 1
    },
    minuteStep: {
      type: Number,
      default: 1
    },
    minDatetime: {
      type: DateTime,
      default: null
    },
    maxDatetime: {
      type: DateTime,
      default: null
    },
    auto: {
      type: Boolean,
      default: false
    },
    weekStart: {
      type: Number,
      default: 1
    },
    flow: {
      type: Array
    },
    title: {
      type: String
    }
  },

  data () {
    const flowManager = this.flow
      ? createFlowManager(this.flow)
      : createFlowManagerFromType(this.type)

    return {
      newDatetime: this.datetime,
      flowManager,
      step: flowManager.first(),
      timePartsTouched: []
    }
  },

  created () {
    document.addEventListener('keydown', this.onKeyDown)
  },

  beforeDestroy () {
    document.removeEventListener('keydown', this.onKeyDown)
  },

  computed: {
    pm () {
      return this.newDatetime.hour > 11;
    },
    nonMilitaryHr () {
      if (this.newDatetime.hour === 0){
        return 12;
      }
      else if (this.newDatetime.hour > 12){
        return this.newDatetime.hour - 12;
      }
      else{
        return this.newDatetime.hour;
      }
    },
    year () {
      return this.newDatetime.year
    },
    month () {
      return this.newDatetime.month
    },
    day () {
      return this.newDatetime.day
    },
    hour () {
      return this.newDatetime.hour
    },
    minute () {
      return this.newDatetime.minute
    },
    dateFormatted () {
      return this.newDatetime.toLocaleString({
        month: 'long',
        day: 'numeric'
      })
    },
    minDatetimeUTC () {
      return this.minDatetime ? this.minDatetime.toUTC() : null
    },
    maxDatetimeUTC () {
      return this.maxDatetime ? this.maxDatetime.toUTC() : null
    },
    minTime () {
      return (
        this.minDatetime &&
        this.minDatetime.year === this.year &&
        this.minDatetime.month === this.month &&
        this.minDatetime.day === this.day
      ) ? this.minDatetime.toFormat('HH:mm') : null
    },
    maxTime () {
      return (
        this.maxDatetime &&
        this.maxDatetime.year === this.year &&
        this.maxDatetime.month === this.month &&
        this.maxDatetime.day === this.day
      ) ? this.maxDatetime.toFormat('HH:mm') : null
    }
  },

  methods: {
    toggleAmPm() {
      let hour = this.newDatetime.hour;
      if (hour < 12){
        hour += 12;
      }
      else{
        hour -= 12;
      }
      this.newDatetime = this.newDatetime.set({ hour })
      this.timePartsTouched['hour'] = true
    },
    hourUp() {
      const hour = this.newDatetime.hour + 1;
      this.newDatetime = this.newDatetime.set({ hour })
      this.timePartsTouched['hour'] = true
    },
    hourDn() {
    const hour = this.newDatetime.hour - 1;
      this.newDatetime = this.newDatetime.set({ hour })
      this.timePartsTouched['hour'] = true
    },
    minUp() {
    const minute = this.newDatetime.minute + 1;
    this.newDatetime = this.newDatetime.set({ minute })
    this.timePartsTouched['minute'] = true
    },
    minDn() {
      const minute = this.newDatetime.minute - 1;
    this.newDatetime = this.newDatetime.set({ minute })
    this.timePartsTouched['minute'] = true
    },
    nextStep () {
      this.$emit('confirm', this.newDatetime)
    },
    showYear () {
      this.step = 'year'
      this.flowManager.diversion('date')
    },
    showMonth () {
      this.step = 'month'
      this.flowManager.diversion('date')
    },
    confirm () {
      this.nextStep()
    },
    cancel () {
      this.$emit('cancel')
    },
    now (){
      const temptime = new Date();
      const minute = temptime.getMinutes();
      const hour = temptime.getHours();
      this.newDatetime = this.newDatetime.set({ minute, hour })
      this.timePartsTouched['minute'] = true
      this.timePartsTouched['hour'] = true

    },
    onChangeYear (year) {
      this.newDatetime = this.newDatetime.set({ year })

      if (this.auto) {
        this.nextStep()
      }
    },
    onChangeMonth (month) {
      this.newDatetime = this.newDatetime.set({ month })

      if (this.auto) {
        this.nextStep()
      }
    },
    onChangeDate (year, month, day) {
      this.newDatetime = this.newDatetime.set({ year, month, day })

      if (this.auto) {
        this.nextStep()
      }
    },
    onKeyDown (event) {
      switch (event.keyCode) {
        case KEY_ESC:
        case KEY_TAB:
          this.cancel()
          break

        case KEY_ENTER:
          this.nextStep()
          break
      }
    }
  }
}
</script>
<style>
@import './pk.font.css';
</style>
<style>
.hrMinTable{
  margin:15px auto 20px
}
.td1{
  padding-right: 50px;
}
.td2{
  padding-right: 40px;
}
.hrMinTxt{
  text-align: center;
  color:#74447A;
}
.ampm{
  color: white;
  background: #74447A;
  -moz-border-radius: 4px;
  -webkit-border-radius: 4px;
  border-radius: 4px;
  padding: 10px 20px;
  }
.caretUpDown{
  border:none;
  background:none;
  color:#74447A;
}
.vdatetime-popup {
  box-sizing: border-box;
  z-index: 1000;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 342px;
  max-width: calc(100% - 30px);
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.3);
  color: #444;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  line-height: 1.18;
  background: #fff;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);

  & * {
    box-sizing: border-box;
  }
}

.vdatetime-popup__header {
  padding: 18px 30px;
  background: #3f51b5;
  color: #fff;
  font-size: 32px;
}

.vdatetime-popup__title {
  margin-bottom: 8px;
  font-size: 21px;
  font-weight: 300;
}

.vdatetime-popup__year {
  font-weight: 300;
  font-size: 14px;
  opacity: 0.7;
  cursor: pointer;
  transition: opacity .3s;

  &:hover {
    opacity: 1;
  }
}

.vdatetime-popup__date {
  line-height: 1;
  cursor: pointer;
}

.vdatetime-popup__actions {
  padding: 0 20px 10px 30px;
  text-align: right;
}

.vdatetime-popup__actions__button {
  display: inline-block;
  border: none;
  padding: 10px 20px;
  background: transparent;
  font-size: 16px;
  cursor: pointer;
  transition: color .3s;

  border: 1px solid #74447A;
  -moz-border-radius: 4px;
  -webkit-border-radius: 4px;
  border-radius: 4px;
}
.vdatetime-popup__actions__button--cancel{
  color: #74447A;
  background: white;
}
.vdatetime-popup__actions__button--confirm{
  color: white;
  background: #74447A;
}
.buttonHover:hover{
    color: white;
    background: #c15f4a;
}
</style>
