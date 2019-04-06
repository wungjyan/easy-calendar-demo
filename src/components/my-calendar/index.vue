<template>
  <div class="my-calendar">
    <!--  头部年月切换-->
    <div class="my-calendar-header">
      <span>
        <i class="iconfont icon-lefttwice" @click="reduceYear" style="margin-right:5px;"></i>
        <i class="iconfont icon-left" @click="reduceMonth"></i>
      </span>
      <span>{{year}}年{{month+1}}月</span>
      <span>
        <i class="iconfont icon-right" @click="addMonth" style="margin-right:5px;"></i>
        <i class="iconfont icon-righttwice" @click="addYear"></i>
      </span>
    </div>
    <!----------------------->

    <!--  主体日期显示-->
    <div class="my-calendar-body">
      <div class="week">
        <span v-for="item in week" :key="item">{{item}}</span>
      </div>
      <ul class="date-wrapper">
        <li v-for="(item,index) in newAllDates" :key="index">
          <span v-for="(innerItem,idx) in item" :key="idx" :class="{'current':innerItem.m==='current','choose':getCurrentTime(innerItem)===hadChooseTime,'today':getCurrentTime(innerItem)===todayTime}" @click="handleClickDate(innerItem)">
            {{innerItem.date}}
          </span>
        </li>
      </ul>
    </div>
    <!----------------------->
  </div>
</template>

<script>
export default {
  name: 'calendar',
  data () {
    return {
      year: '', // 当前显示的年
      month: '', // 当前显示的月
      week: ['日', '一', '二', '三', '四', '五', '六'],
      allShowDates: [], // 当前要显示的所有日期，包括上个月和下个月
      hadChooseTime: 0 // 当前点击的日期时间戳
    }
  },
  computed: {
    // 对所有日期分组，7天一组
    newAllDates () {
      return this.chunk(this.allShowDates, 7)
    },
    // 今天的时间戳
    todayTime () {
      let today = new Date()
      let d = {
        year: today.getFullYear(),
        month: today.getMonth(),
        date: today.getDate()
      }
      return this.getCurrentTime(d)
    }
  },
  methods: {
    // 获取当前月信息
    getCurrentMonthInfo (year, month) {
      // 本月第一天
      let firstDate = new Date(year, month, 1)
      // 本月最后一天
      let lastDate = new Date(year, month + 1, 0)
      return {
        firstDay: firstDate.getDay(), // 本月第一天的星期(0~6)
        lastDay: lastDate.getDay(), // 本月最后一天的星期(0~6)
        days: lastDate.getDate() // 本月最后一天的日期即为本月总天数
      }
    },
    // 获取要展示的日期数组
    // 周日放到最前面
    getAllDates (year, month) {
      // 本月信息
      let thisMonthInfo = this.getCurrentMonthInfo(year, month)
      // 上月信息
      let prevMonthInfo = this.getCurrentMonthInfo(year, month - 1)
      // 当前要展示的日期数组
      let allShowDates = []
      // 向数组中添加当前月日期,m是标记，'current'表示当前月，'prev'表示上个月，'next'表示下个月
      for (let i = 0; i < thisMonthInfo.days; i++) {
        allShowDates.push({
          year: year,
          month: month,
          date: i + 1,
          m: 'current'
        })
      }
      // 向数组头部添加上个月日期
      // thisMonthInfo.firstDay-1 本月第一天的星期减1就是上个月的最后一天的星期，最后一天星期n,则在前添加n天
      // 星期的范围是0～6
      for (let i = 0; i < thisMonthInfo.firstDay; i++) {
        // 如果month为0，即1月，则上月为往年12月，year-1,month为11
        if (month === 0) {
          allShowDates.unshift({
            year: year - 1,
            month: 11,
            date: prevMonthInfo.days - i,
            m: 'prev'
          })
        } else {
          allShowDates.unshift({
            year: year,
            month: month - 1,
            date: prevMonthInfo.days - i,
            m: 'prev'
          })
        }
      }
      // 向数组尾部添加下个月日期
      // 7-thisMonthInfo.lastDay-1 7减去本月最后一个星期再减去周日，就是一周剩余的天数，即是下个月要接的天数
      for (let i = 0; i < 7 - thisMonthInfo.lastDay - 1; i++) {
        // 如果month为11，即12月，则下个月是明年1月，year+1,month为0
        if (month === 11) {
          allShowDates.push({
            year: year + 1,
            month: 0,
            date: i + 1,
            m: 'next'
          })
        } else {
          allShowDates.push({
            year: year,
            month: month + 1,
            date: i + 1,
            m: 'next'
          })
        }
      }
      return allShowDates
    },
    // 减年份
    reduceYear () {
      this.year = this.year - 1
      this.allShowDates = this.getAllDates(this.year, this.month)
    },
    // 减月份
    reduceMonth () {
      this.month = this.month - 1
      // 当month为-1时，年份应该减1，月份是12月
      if (this.month === -1) {
        this.year = this.year - 1
        this.month = 11
      }
      this.allShowDates = this.getAllDates(this.year, this.month)
    },
    // 加年份
    addYear () {
      this.year = this.year + 1
      this.allShowDates = this.getAllDates(this.year, this.month)
    },
    // 加月份
    addMonth () {
      this.month = this.month + 1
      // 当month为12时，年份应该加1，月份为1月
      if (this.month === 12) {
        this.year = this.year + 1
        this.month = 0
      }
      this.allShowDates = this.getAllDates(this.year, this.month)
    },
    // 点击日期
    handleClickDate (d) {
      // 是否需要切换
      if (d.m === 'prev') {
        this.reduceMonth()
      } else if (d.m === 'next') {
        this.addMonth()
      }
      // 计算当前日期的事件戳
      this.hadChooseTime = this.getCurrentTime(d)
      // 派送日期给父元素
      this.$emit('select', d)
      console.log(`${d.year}-${d.month + 1}-${d.date}`)
    },
    // 获取时间戳
    getCurrentTime (d) {
      return new Date(d.year, d.month, d.date).getTime()
    },
    // 数组分块
    chunk (arr, size) {
      return Array.from({ length: Math.ceil(arr.length / size) }, (v, i) =>
        arr.slice(i * size, i * size + size)
      )
    }
  },
  mounted () {
    // 初始化当前日期
    let year = new Date().getFullYear()
    let month = new Date().getMonth()
    console.log(this.getAllDates(year, month))
    this.year = year
    this.month = month
    this.allShowDates = this.getAllDates(year, month)
  }
}
</script>

<style lang="scss" scoped>
.my-calendar {
  .my-calendar-header {
    display: flex;
    justify-content: space-around;
    margin-bottom: 20px;
    padding-top: 10px;
  }
  .my-calendar-body {
    .week {
      width: 100%;
      display: flex;
      justify-content: space-around;
      padding-bottom: 5px;
      border-bottom: 2px solid rgb(235, 238, 245);
      margin-bottom: 5px;
      span {
        width: 24px;
        text-align: center;
      }
    }
    .date-wrapper {
      li {
        display: flex;
        justify-content: space-around;
        margin-bottom: 5px;
        span {
          width: 24px;
          height: 24px;
          border-radius: 50%;
          color: #c0c4cc;
          text-align: center;
          &.current {
            color: #606266;
            &.today {
              color: red;
            }
            &.choose {
              background: #409eff;
              color: #fff;
            }
          }
        }
      }
    }
  }
}
</style>
