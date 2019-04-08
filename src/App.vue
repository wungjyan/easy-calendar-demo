<template>
  <div id="app">
    <my-calendar @select="handleSelectDate"></my-calendar>
    <div style="text-align:center;margin-top:30px;"> 当前选中的日期：{{d.year}}-{{d.month+1}}-{{d.date}}</div>
  </div>
</template>

<script>
import myCalendar from './components/my-calendar'
export default {
  name: 'app',
  data () {
    return {
      d: {
        year: '',
        month: '',
        date: ''
      }
    }
  },
  components: {
    myCalendar
  },
  methods: {
    handleSelectDate (d) {
      this.d = d
    }
  },
  mounted () {
    let today = new Date()
    this.d = {
      year: today.getFullYear(),
      month: today.getMonth(),
      date: today.getDate()
    }
  }
}
</script>
