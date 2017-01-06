<style lang="scss">
.edit-time-components {
  overflow: hidden;
  div, ul, li, p, button {
    padding: 0;
    margin: 0;
    background: none;
    border: none;
    outline: none;
  }

  .year-month-box {
    text-align: center;
    .year-month {
      display: inline-block;
      font-size: 20px;
      color: #7b7777;
    }
    .pre-month, .next-month {
      display: inline-block;
      color: #7b7777;
      cursor: pointer;
      transform: scaleY(1.5);
      height: 13px;
      line-height: 6px;
      vertical-align: middle;
      padding: 0 5px;
      &:hover {
        color: #2494f2;
      }
    }
  }

  .time-box {
    width: 242px;
    text-align: center;
    float: left;
    margin: 5px;
    border: 1px solid #ddd;
    overflow: hidden;
  }

  .li-item {
    display: inline-block;
    width: 14%;
    font-size: 15px;
    float: left;
    padding: 5px;
    border-radius: 50%;
  }

  .week {
    color: #b5b5b5;
    padding: 0;
  }

  .day {
    cursor: pointer;
  }

  .day:hover {
    background: #ddd;
  }

  .day-false {
    color: rgba(57, 57, 57, .3);
    cursor: not-allowed;
  }

  .day-false:hover {
    background: none;
  }

  .ul-box {
    width: 100%;
  }
}
</style>

<template>
  <div class='edit-time-components'>
    <div id='{{index}}' class='time-box' v-for='(index, item) in datelist'>
      <div class='year-month-box'>
        <p v-if='item.switch' class='pre-month' @click='changeMonth(-1, index)'>&lt;</p>
        <p class='year-month'>{{item.year + '年' + item.month + '月'}}</p>
        <p v-if='item.switch' class='next-month' @click='changeMonth(1, index)'>&gt;</p>
      </div>
      <ul class='ul-box'>
        <li class='li-item week'>周日</li>
        <li class='li-item week'>周一</li>
        <li class='li-item week'>周二</li>
        <li class='li-item week'>周三</li>
        <li class='li-item week'>周四</li>
        <li class='li-item week'>周五</li>
        <li class='li-item week'>周六</li>
      </ul>
      <ul class='ul-box'>
        <li class='li-item' v-for='i in item.week'></li>
        <button
          class='{{"li-item day " + (options.disable.indexOf(formatDate(item.year, item.month, i + 1)) !== -1 ? "day-false" : "")}}'
          :style = '{
            background: (options.selectDay.indexOf(formatDate(item.year, item.month, i + 1)) !== -1 ? "#5bc0de" : ""),
            color: (options.selectDay.indexOf(formatDate(item.year, item.month, i + 1)) !== -1 ? "#fff" : ""),
          }'
          data-index = '{{i + 1}}'
          select = '{{(options.selectDay.indexOf(formatDate(item.year, item.month, i + 1)) !== -1 ? "true" : "false")}}'
          v-for='i in item.monthDay'
          @click='dateClick($event, item, i + 1)'
          @dblclick='dbDateClick($event, item, i + 1)'
          @mouseover='mouseover(item, i + 1)'>
          {{i + 1}}
        </button>
      </ul>
    </div>
  </div>
</template>

<script type="text/javascript">
/*
组件功能:
1.可选择月份展开显示或者作为一个时间框显示
2.可单击选中，支持多选
3.可双击启动连续选择，支持正向，逆向和跳跃不可选日期
4.可原地切换日期
*/
let vm = null
let pageParams = {}
let DEFAULT_DATALIST = [
  {
    year: undefined, // 日期初始年, 默认当前年
    month: undefined, // 日期初始月, 默认当前月
    multiSelect: true, // 日期是否可多选(功能暂未实现)
    switch: false // 当前日期框是否支持切换年月份(功能暂未实现)
  }
]
let DEFAULT_OPTIONS = {
  disable: [], // 不可选日期，格式: '2016-01-01'
  // enable: [], // 可选日期，格式: '2016-01-01'，enable和disable只能有一个，如果都有默认用enable
  selectDay: [], // 已选择的day，格式: '2016-01-01'
  callback: undefined // 点击日期回调函数, callback(selectDateList)
}

export default {
  props: {
    datelist: {
      type: Array,
      required: false,
      default: function () {
        return DEFAULT_DATALIST
      }
    },

    options: {
      type: Object,
      default: function() {
        return DEFAULT_OPTIONS
      },
      coerce: function(val) {
        val.disable = val.disable || []
        val.selectDay = val.selectDay || []
        return val
      }
    }
  },

  methods: {
    changeMonth(type, index) {
      console.log(type, index)
      let date = JSON.parse(JSON.stringify(vm.datelist[index]))
      let year = parseInt(date.year)
      let month = parseInt(date.month) + parseInt(type)
      if(month < 0) {
        year -= 1
        month = 12
      }else if(month > 12) {
        year += 1
        month = 1
      }
      date.year = year
      date.month = month
      vm.getDayConfig(date)
      if(vm.options.enable) {
        vm.options.disable = vm.getDisableDate(year, month, vm.options.enable)
      }
      vm.datelist.$set(index, date)
    },

    formatDate(year, month, day) {
      month = (month + '').length === 2 ? month : '0' + month
      day = (day + '').length === 2 ? day : '0' + day
      return year + '-' + month + '-' + day
    },

    getDisableDate(year, month, enable) {
      let disable = []
      month = (month + '').length === 2 ? month : '0' + month
      for(let i = 1; i < 32; ++i) {
        let day = (i + '').length === 2 ? i : '0' + i
        let date = year + '-' + month + '-' + day
        if(enable.indexOf(date) === -1) {
          disable.push(date)
        }
      }
      return disable
    },

    init() {
      pageParams = {
        isdbclick: false,
        domId: null,
        day: null,
        dom: new Array(33),
        selectDay: new Set()
      }

      for(let item of this.options.selectDay) {
        pageParams.selectDay.add(item)
      }

      let tempList = JSON.parse(JSON.stringify(this.datelist))
      tempList = tempList.length === 0 ? [] : tempList
      tempList.forEach((item, index) => {
        item.id = index
        vm.getDayConfig(item)
      })
      if(vm.options.enable) {
        vm.options.disable = vm.getDisableDate(tempList[0].year, tempList[0].month, vm.options.enable)
      }
      this.datelist = tempList
    },

    getDayConfig(item) {
      let year = item.year || (new Date()).getFullYear()
      let month = item.month || (parseInt((new Date()).getMonth()) + 1)
      let monthDay = (new Date(year, month, 0)).getDate()
      let week = (new Date(year, parseInt(month - 1), 1)).getDay()

      item.year = year
      item.month = month
      item.monthDay = monthDay
      item.week = week
    },

    getdbClickData(item) {
      for(let i = 1; i <= item.monthDay; ++i) {
        let dom = pageParams.dom[i]
        let tMonth = (item.month + '').length === 2 ? item.month : '0' + item.month
        let tDay = (i + '').length === 2 ? i : '0' + i
        let date = item.year + '-' + tMonth + '-' + tDay
        if(dom.style.background !== '') {
          dom.setAttribute('select', 'true')
          pageParams.selectDay.add(date)
        }
      }
    },

    cleardbClick() {
      for(let i = 1; i <= pageParams.dom.length; ++i) {
        let dom = pageParams.dom[i]
        if(!dom) continue
        if(dom.getAttribute('select') !== 'true') {
          dom.style.background = ''
          dom.style.color = '#000'
        }
      }
    },

    dateClick(event, item, day) {
      // console.log(event, item, day)
      if(vm.options.disable.indexOf(vm.formatDate(item.year, item.month, day)) !== -1) return

      if(pageParams.isdbclick) {
        if(item.id === pageParams.domId) {
          pageParams.isdbclick = false
          vm.getdbClickData(item)
          if(vm.options.callback) {
            vm.options.callback([...pageParams.selectDay])
          }
          return
        }else {
          vm.cleardbClick()
        }
      }

      pageParams.isdbclick = false
      let tMonth = (item.month + '').length === 2 ? item.month : '0' + item.month
      let tDay = (day + '').length === 2 ? day : '0' + day
      let date = item.year + '-' + tMonth + '-' + tDay
      if(event.target.getAttribute('select') !== 'true') {
        event.target.style.background = '#5bc0de'
        event.target.style.color = '#fff'
        event.target.setAttribute('select', 'true')
        pageParams.selectDay.add(date)
      }else {
        event.target.style.background = ''
        event.target.style.color = '#000'
        event.target.setAttribute('select', 'false')
        pageParams.selectDay.delete(date)
      }
      if(vm.options.callback) {
        vm.options.callback([...pageParams.selectDay])
      }
    },

    dbDateClick(event, item, day) {
      // console.log(item, day)
      if(vm.options.disable.indexOf(vm.formatDate(item.year, item.month, day)) !== -1) return

      event.target.style.background = '#5bc0de'
      event.target.style.color = '#fff'
      let id = item.id
      pageParams.isdbclick = true
      pageParams.day = day
      if(pageParams.domId === id) {
        return
      }
      pageParams.domId = item.id
      let dom = document.getElementById(id + '').getElementsByClassName('day')
      for(let i = 0; i < dom.length; ++i) {
        let item = dom[i]
        pageParams.dom[item.dataset.index] = item
      }
    },

    mouseover(item, day) {
      if(!pageParams.isdbclick || !pageParams.day) return
      if(pageParams.domId !== item.id) return

      let left = day > pageParams.day ? pageParams.day : day
      let right = day > pageParams.day ? day : pageParams.day
      for(let i = 1; i <= item.monthDay; ++i) {
        if(vm.options.disable.indexOf(vm.formatDate(item.year, item.month, i)) !== -1) continue

        let dom = pageParams.dom[i]
        if(left <= i && i <= right) {
          dom.style.background = '#5bc0de'
          dom.style.color = '#fff'
        }else if(dom.getAttribute('select') !== 'true') {
          dom.style.background = ''
          dom.style.color = '#000'
        }
      }
    }
  },

  watch: {
    datelist: function(val, oldVal) {
      if(JSON.stringify(val) === JSON.stringify(oldVal)) {
        return
      }
      this.init()
    }
  },

  created: function() {
    vm = this
    this.init()
  }
}
</script>
