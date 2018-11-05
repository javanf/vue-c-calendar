<template>
  <c-calendar
    :labels="labels"
    :show-amount="3"
    :start-date="depDate"
    :end-date="returnDate"
    :disable-after="disableAfter"
    :is-same="true"
    @complete="selctDateComplete"
  ></c-calendar>
</template>

<script>
  import moment from 'moment'
  import cCalendar from './calendar.vue'

  export default {
    data () {
      return {
        labels: ['出发', '到达'],
        returnDate: ''
      }
    },
    computed: {
      depDate() {
        return moment().format('YYYY-MM-DD')
      },
      disableAfter () {
        return moment().add(60, 'days').format('YYYY-MM-DD')
      }
    },
    methods: {
      selctDateComplete(startDate, endDate){
        console.log(startDate, endDate);
      }
    },
    components: {
      cCalendar: cCalendar
    }
  }
</script>