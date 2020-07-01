<template>
  <div class="hello">
    <!-- <el-date-picker size='small' v-model="selectDate" type="month"
      placeholder="选择月" value-format="yyyy-MM">
    </el-date-picker>
    <el-button size='small' @click='changeDate'>确定</el-button> -->
    <full-calendar  
      :config="config" 
      :events="events"
      ref="calendar" 
      @event-selected='eventClick' 
      @day-click="dayClick">
    </full-calendar> 
    <add-schedule v-if="isAdd" :isAdd='isAdd' :editItem='editItem' @add='addItem' @close='isAdd = false'></add-schedule>
  </div>
</template>
<script>
import { FullCalendar } from 'vue-full-calendar'
import 'fullcalendar/dist/fullcalendar.css'
import addSchedule from '@/components/calendar/add.vue'

export default {
   data () {
    return {
      isAdd:false,
      selectDate:'',//日期选择器选中的月份
      config: {
        firstDay:'0',//以周日为每周的第一天
        // weekends: true,//是否在日历中显示周末
        locale: 'zh-cn',//语言
        defaultView: 'month',//默认按月显示
        height: 'auto',//高度
        fixedWeekCount:false,//是否固定显示六周
        // weekMode:"liquid",//周数不定，每周的高度可变，整个日历高度不变
        allDaySlot:false,
        // allDay:true,
        header: {//表头信息
          left: 'prev, next, today',
          center: 'title',
          right: 'hide, custom'
        },
      },
      events: [
        {
        id:1,
        title:'项目1：4小时',
        start:'2020-07-03',
        end:'2020-07-03',
        color: "#c1e9fe"
      },
      {
        id:2,
        title:'项目2：4小时',
        start:'2020-07-03',
        end:'2020-07-03',
        color: "#c1e9fe"
      }, {
        id:3,
        title:'项目1：8小时',
        start:'2020-07-04',
        end:'2020-07-04',
        color: "#c1e9fe"
      }
      ],
      newItem:{},
      editItem:{}
    }
  },
  components : { FullCalendar, addSchedule },
  methods:{
    changeDate(){
      // this.$refs.calendar.fireMethod('gotoDate', this.selectDate)
      this.$refs.calendar.fireMethod('prev');
    },
    eventClick(event){ //events的点击事件
      this.editItem = event
      this.isAdd = true
    },
    dayClick(date, jsEvent, view){ //日期的点击事件
      this.editItem = {
        time: this.getDateTime(date._i)
      }
      this.isAdd = true
    },
    addItem(detail){
      this.newItem = JSON.parse(detail)
      if(this.editItem.id){//如果是编辑，就删掉该条
        this.events.forEach( (el,ind)=>{
          if(el.id == this.editItem.id){
            this.events.splice(ind,1)
          }
        })
      }
      let len = this.getDaysBetween(this.newItem.period[0], this.newItem.period[1])
      if (len) {
        for(let i = 0; i<len ; i++) {
          this.events.push({
            id : this.editItem.id?this.editItem.id:this.setUuid(),
            title : this.newItem.title,
            start : i===0? this.newItem.period[0]:this.getNextDate(this.newItem.period[0], i),
            end : i===0? this.newItem.period[0]:this.getNextDate(this.newItem.period[0], i),
            color: "#c1e9fe"
          })
        }
      } else {
        this.events.push({
            id : this.editItem.id?this.editItem.id:this.setUuid(),
            title : this.newItem.title,
            start : this.newItem.period[0],
            end : this.newItem.period[0],
            color: "#c1e9fe"
          })
      }
      
    },
    getDateTime(timestamp) {
    let dateTime = '';
    if (timestamp) {
        let time = new Date(timestamp),
            timeYear = time.getFullYear(),
            timeMonth = time.getMonth() + 1,
            timeDate = time.getDate();
        dateTime = timeYear + '-' + format(timeMonth) + '-' + format(timeDate);
    }
    return dateTime;
    // 数据格式
    function format(number) {
        return number < 10 ? '0' + number : number;
    }},
    getNextDate(date, day) {
      let dd = new Date(date);
      dd.setDate(dd.getDate() + day);
      let y = dd.getFullYear();
      let m = dd.getMonth() + 1 < 10 ? "0" + (dd.getMonth() + 1) : dd.getMonth() + 1;
      let d = dd.getDate() < 10 ? "0" + dd.getDate() : dd.getDate();
      return y + "-" + m + "-" + d;
    },
    getDaysBetween(dateString1,dateString2) {
      let  startDate = Date.parse(dateString1);
      let  endDate = Date.parse(dateString2);
      let days=(endDate - startDate)/(1*24*60*60*1000);
      return  days;
    },
    setUuid(){
        let s = [];
        let hexDigits = "0123456789abcdef";
        for(let i = 0; i < 36; i++){ s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1); }
        s[14] = "4";  
        s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); 
        s[8] = s[13] = s[18] = s[23];
        let uuid = s.join("");
        return uuid;
    },
   
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.fc-view-container *, .fc-view-container *:before, .fc-view-container *:after {
    color: #999;
}
.fc-title {
  word-break:normal;
  width:auto;
  display:block;
  white-space:pre-wrap;
  word-wrap : break-word ;
  overflow: hidden ;
}
</style>
