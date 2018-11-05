<template>
  <div id="calerdar">
    <div>
      <div id="calendar-date" v-bind:class="{single: labels.length === 1}">
        <div class="date fl">
          <span class="year">{{startDateMomentYear}}</span>
          <span class="month-date">{{startDateMomentMonth}}</span>
          <span class="label">{{labels[0]}}</span>
        </div>
        <div class="date fr" v-if="endDateMoment">
          <span class="year">{{endDateMomentYear}}</span>
          <span class="month-date">{{endDateMomentMonth}}</span>
          <span class="label">{{labels[1]}}</span>
        </div>
        <div class="date placeholder fr" v-else=""></div>
        <div class="spliter"></div>
      </div>
      <ul id="week-label">
        <li class="item" v-for="(week, windex) in weeks" :key="windex">{{week}}</li>
      </ul>
    </div>
    <div>
      <div id="month-list" v-bind:state="waiting ? 'waiting' : 'complete' " v-on:scroll="loadRepeat">
        <section class="month-item" v-for="(month, mindex) in monthList" :key="mindex">
          <p class="month-info">{{month.numStr}}月</p>
          <ul class="month-main">
            <li class="item null" v-for="(pmitem, pmindex) in month.prefix" :key="pmindex"></li>
            <li class="item date" @click="checkDate(date)" v-for="(date, dindex) in month.dateList" :key="dindex" :class="{'disable': date.isDisable,'today': date.isToday,'start-date': date.isStartDate, 'end-date': date.isEndDate, 'progress': date.isProgress}">
              <span class="num">{{date.dateData ? date.dateData.date : ''}}</span>
            </li>
            <li class="item null" v-for="(smitem, smindex) in month.surfix" :key="smindex"></li>
          </ul>
        </section>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import moment from 'moment'

var TODAY = moment().startOf('date')
var SCROLL_LIMIT_PERCENT = 0.5
var MONTH_CH = ['一', '二', '三', '四', '五', '六', '七', '八', '九', '十', '十一', '十二']

var DateItem
var MonthItem
var makeDateList = function (data) {
  var firstDateInMonth = data.curMonth.clone()
  var lastDateInMonth = data.curMonth.clone().endOf('month')
  var prefixAmount = firstDateInMonth.day()
  var contentAmount = lastDateInMonth.date()
  var surfixAmount = (6 - lastDateInMonth.day()) % 6
  var result = []
  var i, l

  for (i = 0, l = prefixAmount + contentAmount + surfixAmount; i < l; i += 1) {
    if (i < prefixAmount || i >= prefixAmount + contentAmount) {
      result.push(new DateItem())
    } else {
      result.push(new DateItem(firstDateInMonth.clone(), data.disableBeforeMoment, data.disableAfterMoment))
      firstDateInMonth.add(1, 'd')
    }
  }
  return result
}
var formatDate = function (date, format) {
  return moment(date).format(format || 'YYYY-MM-DD')
}

DateItem = function (date, disableBeforeMoment, disableAfterMoment) {
  if (date !== undefined) {
    this.dateData = {
      year: date.year(),
      month: date.month(),
      date: date.date()
    }
    this.timeStamp = date.toDate().getTime()
    this.isToday = date.isSame(TODAY)
    this.isDisable = (disableBeforeMoment && date.isBefore(disableBeforeMoment)) || (disableAfterMoment && date.isAfter(disableAfterMoment))
  } else {
    this.isNull = true
  }
  this.isStartDate = false
  this.isEndDate = false
  this.isProgress = false
}
MonthItem = function (data) {
  var targetMonth = data.curMonth.clone().startOf('month')
  this.num = targetMonth.month()
  this.numStr = MONTH_CH[this.num]
  this.dateList = makeDateList(data)
}

export default {
  name: "CCalendar",
  props: {
    labels: {
      type: Array,
      required: true
    },
    isSame: { // 是否能相同
      type: Boolean,
      default: false
    },
    startDate: String,
    endDate: String,
    showAmount: {
      type: Number,
      default: 3
    },
    disableBefore: {
      type: String,
      default: formatDate(TODAY)
    },
    disableAfter: String,
    start: String,
    sameEnable: {
      type: Boolean,
      default: true
    }
  },
  onShow () {
    this.animateFreeze = false
  },
  data () {
    var startMonth = moment(this.start).startOf('month')
    var singleMode = this.labels.length === 1
    return {
      weeks: ['日', '一', '二', '三', '四', '五', '六'],
      disableBeforeMoment: moment(this.disableBefore),
      disableAfterMoment: this.disableAfter && moment(this.disableAfter),
      firstMonth: startMonth,
      curMonth: startMonth.clone(),
      curAmount: 0,
      monthList: [],
      startDateMoment: this.startDate && moment(this.startDate),
      singleMode: singleMode,
      endDateMoment: !singleMode && this.endDate && moment(this.endDate),
      loadFreeze: false,
      animateFreeze: false,
      waiting: false
    }
  },
  computed: {
    startDateMomentYear () {
      return this.startDateMoment.year()
    },
    startDateMomentMonth () {
      return this.startDateMoment.format('MM-DD')
    },
    endDateMomentYear () {
      return this.endDateMoment && this.endDateMoment.year()
    },
    endDateMomentMonth () {
      return this.endDateMoment && this.endDateMoment.format('MM-DD')
    }
  },
  watch: {
    startDateMoment: 'refreshSelectRagne',
    endDateMoment: 'refreshSelectRagne',
    monthList: 'refreshSelectRagne'
  },
  methods: {
    addMonth () {
      var monthItem = new MonthItem({
        curMonth: this.curMonth,
        disableBeforeMoment: this.disableBeforeMoment,
        disableAfterMoment: this.disableAfterMoment,
        startDateMoment: this.startDateMoment,
        endDateMoment: this.endDateMoment
      })
      this.monthList.push(monthItem)
      this.curAmount += 1
      this.curMonth.add(1, 'months')
    },
    checkScroll () {
      var wrapper = this.calendarWrap
      var scrollHeight = wrapper.scrollHeight
      var viewHeight = wrapper.clientHeight
      return scrollHeight - wrapper.scrollTop - viewHeight < scrollHeight * SCROLL_LIMIT_PERCENT
    },
    loadRepeat () {
      var self = this
      if (!self.loadFreeze && self.showAmount > self.curAmount) {
        self.loadFreeze = true
        self.addMonth()
        setTimeout(() => {
          self.loadFreeze = false
          self.loadRepeat()
        }, 20)
      }
    },
    refreshSelectRagne () {
      var startTimeStamp = this.startDateMoment && this.startDateMoment.toDate().getTime()
      var endTimeStamp = !this.singleMode && this.endDateMoment && this.endDateMoment.toDate().getTime()
      var prefDate = {}
      this.monthList.forEach((monthData) => {
        monthData.dateList.forEach((dateData) => {
          var dateTimeStamp = dateData.timeStamp
          if (dateData.isNull) {
            dateData.isStartDate = dateData.isEndDate = false
            dateData.isProgress = (prefDate.isProgress || prefDate.isStartDate) && endTimeStamp
          } else {
            dateData.isStartDate = dateTimeStamp === startTimeStamp
            dateData.isEndDate = dateTimeStamp === endTimeStamp
            dateData.isProgress = dateTimeStamp > startTimeStamp && dateTimeStamp < endTimeStamp
          }
          prefDate = dateData
        })
      })
    },
    checkDate (date) {
      if (date.isDisable || date.isNull || this.animateFreeze) {
        return
      }
      if (this.startDateMoment && (this.singleMode || this.endDateMoment)) {
        this.startDateMoment = this.endDateMoment = null
      }
      var dateData = date.dateData
      var checkTargetMoment = moment([dateData.year, dateData.month, dateData.date])
      if (!this.startDateMoment || checkTargetMoment[!this.sameEnable ? 'isSameOrBefore' : 'isBefore'](this.startDateMoment)) {
        this.startDateMoment = checkTargetMoment
      } else {
        if (!this.isSame && '' + checkTargetMoment + '' === '' + this.startDateMoment + '') {
          console.log('入离时间不能相同')
          return
        }
        this.endDateMoment = checkTargetMoment
      }
      if (this.singleMode || this.endDateMoment) {
        this.confirm()
        this.waiting = false
      } else {
        this.waiting = true
      }
    },
    confirm () {
      var startDate = formatDate(this.startDateMoment)
      var endDate = !this.singleMode ? formatDate(this.endDateMoment) : undefined
      this.animateFreeze = true
      this.$emit('complete', startDate, endDate)
    }
  },
  mounted () {
    Vue.nextTick(() => {
      // this.calendarWrap = this.$el.querySelector('#month-list')
      // this.$el.setAttribute('calendar-type', this.singleMode ? 'single' : 'multiple')
      this.loadRepeat()
    })
  }
}
</script>

<style lang="scss">
#calerdar{
    background: #fff;
    ul,li{ 
      padding:0;
      margin:0;
      list-style:none
    }
    .layout-page-main{
        border-top:px2rpx(1) solid #E2E2E2;
    }
    #calendar-date{
        height: 4.2rem;
        overflow: hidden;
        .date{
            padding: 0.31rem 0;
            text-align: center;
            width: 10rem;
            .year{
                display: block;
                line-height: 1rem;
                font-size: 1rem;
                color: #9b9b9b;
            }
            .month-date{
                display: block;
                line-height: 1.5rem;
                font-size: 1.5rem;
                color: #3b4f62;
            }
            .label{
                display: block;
                line-height: 1rem;
                font-size: 1rem;
                color: #9b9b9b;
            }
            &.fl{
                float: left;
            }
            &.fr{
                float: right;
            }
            &.placeholder{
                height: 100%;
                position: relative;
                &::before{
                    content: '';
                    display: block;
                    position: absolute;
                    left: 50%;
                    top: 50%;
                    height: 1px;
                    width: 0.9rem;
                    background-color: gray;
                }
            }
        }
        .spliter{
            $block-height: 2rem;
            height: $block-height;
            margin: 0.7rem auto;
            position: relative;
            overflow: hidden;
            &::before{
                content: '';
                position:absolute;
                display: block;
                height: $block-height;
                width: 1px;
                background-color: gray;
                left: 45%;
                transform: rotate(32deg);
            }
        }
    }
    #week-label{
        $height: 2rem;
        overflow: hidden;
        border-bottom: 1px solid #dedede;
        .item{
            float: left;
            width: (100%/7);
            text-align: center;
            $height: $height;
            line-height: $height;
            color: #b8b8b8;
            font-size: 1.2rem;
            &:first-child,
            &:last-child{
                color: #ff7362;
            }
        }
    }
    $item-width: (10rem / 7);
    $num-width: 2rem;
    $space-w: ($item-width - $num-width) * 0.5;
    #month-list{
        padding-top: 0.2rem;
        height: 100%;
        overflow: auto;
        .month-item{
            overflow: hidden;
            padding-top: 0.35rem;
            .month-info{
                line-height: 0.86rem;
                padding-left: 0.37rem;
                color: #b7b7b7;
                font-size: 1.5rem;
                text-align: left;
            }
            .month-main{
                overflow: hidden;
                padding-top: 0.2rem;
                .item{
                    float: left;
                    width: (100%/7);
                    color: #969696;
                    .num{
                        display:block;
                        position: relative;
                        // margin: auto;
                        height: $num-width;
                        line-height: $num-width;
                        text-align: center;
                        font-size: 1rem;
                    }
                    &.disable{
                        color: #e2e2e2;
                    }
                    &.today{
                      color: #0cc071;
                    }
                    &.start-date .num{
                      border-radius: .2rem 0 0 .2rem;
                      background-color: #0cc071;
                      color: #fff;
                    }
                    &.end-date .num{
                        border-radius: 0 .2rem .2rem 0;
                        background-color: #0cc071;
                        color: #fff;
                    }
                    &.start-date{
                        &.end-date .num{
                          border-radius: .2rem;
                        }
                    }
                    &.progress .num{
                      background-color: #0cc0717d;
                      color: #fff;
                    }
                }
            }
        }
    }
    $space-w: 1rem;
    &[calendar-type="multiple"]{
        [state="complete"]{
            .start-date .num{
                margin-left: $space-w;
                padding-right: $space-w;
                border-top-left-radius: 99rem;
                border-bottom-left-radius: 99rem;
            }
            .end-date .num{
                margin-right: $space-w;
                padding-left: $space-w;
                border-top-right-radius: 99rem;
                border-bottom-right-radius: 99rem;
            }
        }
        [state="waiting"]{
            .start-date .num{
                margin: 0 $space-w;
                border-radius: 100%;
            }
        }
    } 
    &[calendar-type="single"]{
        // set-header-height(1.3rem)
        #week-label{
            margin-top: 0.3rem;
        }
        .item.start-date .num{
            margin: 0 $space-w;
            padding: 0;
            border-radius: 100%;
        }
    }
}
</style>
