<template>
  <div class="calendar">
    <div class="calendar-container">

      <div class="calendar-title">
        <span class="el-icon-arrow-left" @click.stop="previousMonth()"></span>
        <span class="year-month">{{currentYear + '年' + currentMonth + '月'}}</span>
        <span class="el-icon-arrow-right" @click.stop="nextMonth()"></span>
      </div>

      <div class="calendar-week">
        <span class="week-name" v-for="(name, n) in weekNameList" :key="name + '-' + n">{{name}}</span>
      </div>

      <div class="calendar-day-container">
        <div 
          v-for="(day, i) in aftreSetDayList"
          :key="day + '-' + i"
          :class="makeColumClass(day, i)"
          class="calendar-day">
          <span class="day">{{returnDay(day)}}</span>
          <div class="am-box">
            <div v-if="i%7===0" class="noon-box">
              <span class="am-span">上午</span>
            </div>
            <div class="am-input-box" v-if="day">
              <p
                v-for="(serial,index) in amSerialList"
                :key="'am-input-'+serial"
                :id="'am-input-'+serial"
                class="event-input"
                :style="'background-color:'+ 
                  calendarData[setMark(day,0,index)].color +';' +
                  'border:'+calendarData[setMark(day,0,index)].border+';'"
                :data-week="weekNameList[i]"
                :data-mark="setMark(day,0,index)"
                :data-id="calendarData[setMark(day,0,index)].eventId"
                @mouseover="mouseover"
                @mouseout="mouseout"
                >
                <input 
                  type="text" 
                  :title="calendarData[setMark(day,0,index)].label"
                  :maxlength="maxlength"
                  v-model.trim="calendarData[setMark(day,0,index)].label"
                  @keyup="inputEvent"
                  @focus.self="focus"
                  @blur.self="blur"
                  :placeholder="calendarData[setMark(day,0,index)].placeholder"/>
                <i 
                  v-show="calendarData[setMark(day,0,index)].canClear && !showSelectorPanel"
                  class="el-icon-circle-close"
                  :class="calendarData[setMark(day,0,index)].color?'white':'gray'" 
                  @click.stop="clearEventContent">    
                </i>
                <em 
                  @click.stop="colotBtnClick"
                  :style="'background-color:'+ (calendarData[setMark(day,0,index)].color || defaultEventBgColor) +';'">
                </em>
              </p>
            </div>
          </div>
          <div class="pm-box">
            <div v-if="i%7==0" class="noon-box">
              <span class="pm-span">下午</span>
            </div>
            <div class="pm-input-box" v-if="day">
              <p
                v-for="(serial,index) in pmSerialList"
                :key="'pm-input-'+serial"
                :id="'pm-input-'+serial"
                class="event-input"
                :style="'background-color:'+ 
                  calendarData[setMark(day,1,index)].color +';' +
                  'border:'+calendarData[setMark(day,1,index)].border+';'"
                :data-week="weekNameList[i]"
                :data-mark="setMark(day,1,index)"
                :data-id="calendarData[setMark(day,1,index)].eventId"
                @mouseover="mouseover"
                @mouseout="mouseout"
                >
                <input 
                  type="text" 
                  :title="calendarData[setMark(day,1,index)].label"
                  :maxlength="maxlength"
                  v-model.trim="calendarData[setMark(day,1,index)].label"
                  @keyup="inputEvent"
                  @focus.self="focus"
                  @blur.self="blur"
                  :placeholder="calendarData[setMark(day,1,index)].placeholder">
                <i 
                  v-show="calendarData[setMark(day,1,index)].canClear && !showSelectorPanel"
                  class="el-icon-circle-close" 
                  :class="calendarData[setMark(day,0,index)].color?'white':'gray'" 
                  @click.stop="clearEventContent">    
                </i>
                <em 
                  @click.stop="colotBtnClick"
                  :style="'background-color:'+ (calendarData[setMark(day,1,index)].color || defaultEventBgColor) +';'">
                </em>
              </p>
            </div>
          </div>
        </div>
        <!-- 颜色选择面板部分 ================= strat ================ -->
          <selector-panel 
            v-if="showSelectorPanel" 
            :show="showSelectorPanel" 
            :data="selectorPanelData" 
            :configData="selectorPanelConfig" 
            @optionClick="optionClick" 
            @close="selectorPanelColose"
            :targetElementMark="targetElementMark">
          </selector-panel>
        <!-- 颜色选择面板部分 ================= end ================ -->
        <!-- 颜色选择面板部分 ================= strat ================ -->
          <color-panel 
            v-if="showColorPanel" 
            :show="showColorPanel" 
            :colorList="colorList" 
            :configData="colorPanelConfig" 
            @colorItemClick="colorItemClick" 
            @close="colorPanelColose"
            :targetElementMark="targetElementMark">
          </color-panel>
        <!-- 颜色选择面板部分 ================= end ================ -->
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import colorPanel from './colorPanel.vue'
  import selectorPanel from './selectorPanel.vue'
  import axios from "axios";
  export default{
    name: 'calendar',
    components: {colorPanel, selectorPanel},
    props: {
      teamId: null,
    },
    computed: {
      // // 每月有几个7天
      returnRows() {
        if (this._isLeapYear(this.currentYear)) {this.dayLengthList.splice(1,1,29);}
        else {this.dayLengthList.splice(1,1,28);}
        // 每月的1号是周几
        let total = this.dayLengthList[this.currentMonth-1] + this.weekFirstDayOfMonth;
        return Math.ceil(total/this.weekNameList.length);
      },
      // 设置后的dayList
      aftreSetDayList() {
        this.dayList = [];
        let length = this.returnRows * 7;
        for (let i = 0; i < length; i++) {
          if (i < this.weekFirstDayOfMonth || (i-this.weekFirstDayOfMonth) >= this.dayLengthList[this.currentMonth-1]) {
            this.dayList.push('');
          } else {
            this.dayList.push(this.currentYear+'-'+this.currentMonth+'-'+((i+1)-this.weekFirstDayOfMonth<10?'0'+((i+1)-this.weekFirstDayOfMonth):(i+1)-this.weekFirstDayOfMonth));
          }
        }
        this._setDefaultCalendarData();
        return this.dayList;
      }
    },
    data(){
      return {
        dayList: [], // 渲染日期格的数组
        amSerialList: ['first','second','third'], // 上午事件序号list
        pmSerialList: ['first','second','third'], // 下午事件序号list
        currentYear: new Date().toJSON().split('-')[0], // 当前年
        currentMonth: new Date().toJSON().split('-')[1], // 当前月
        weekFirstDayOfMonth: this._witchDayOfTheWeek(new Date().toJSON().split('-')[0], new Date().toJSON().split('-')[1], '01'), // 某月的一号是周几
        weekNameList: ['周日','周一','周二','周三','周四','周五','周六'], // 星期几的数组
        dayLengthList: [31,28,31,30,31,30,31,31,30,31,30,31], // 12个月每月天数
        maxlength: 20, // input最大字符数
        inputBorder: '1px solid #fff',
        inputDefaultBorder: '1px dotted transparent', // input父元素默认边框样式
        inputHoverBorder: '1px dotted #ebeef5', // input父元素hover时的边框样式
        defaultEventBgColor: '#8fcfff',
        defaultContent: '新建事件',
        showColorPanel: false, // 是否显示颜色选择面板
        colorList: ['#8fcfff','#abdafc','#a0eaea','#b2efef','#f3b0b0','#f8bcbc','#fac5a7','#fcdac2','#dbdbdb'], // 颜色选择面板颜色list
        targetElementMark: '', // 被点击的颜色面板按钮所在的父元素
        colorPanelConfig: {
          containerStyle: `position:absolute;width:45px;height:40px;padding-top:5px;
                           display:flex;flex-wrap:wrap;background-color:#fff;-webkit-box-shadow:0 0 5px #ddd;
                           -moz-box-shadow:0 0 5px #ddd;-ms-box-shadow:0 0 5px #ddd;box-shadow:0 0 5px #ddd;`,
          itemStyle: `flex:none;width:8px;max-width:8px;min-width:8px;height:8px;margin-left:5px;cursor:pointer;
                      -webkit-border-radius:2px;-moz-border-radius:2px;-ms-border-radius:2px;border-radius:2px;`,
          offsetParentClass: 'calendar-day-container', // 组价顶层的已定位的父元素的class
        }, // 颜色面板配置参数
        selectorTimer: null, // 选择面板延时定时器序号
        showSelectorPanel: false, // 选择面板开关
        selectorPanelData: {
          optionList: [], // option的list
          panelHeight: 100, // 面板高度
        }, // 选择面板数据
        selectorPanelConfig: {
          offsetParentClass: 'calendar-day-container', // 组价顶层的已定位的父元素的class
        }, // 选择面板配置数据
        calendarData: {},
      }
    },
    mounted(){

    },
    methods: {
      // 模糊搜索事件名
      searchEventName(label, callback) {
        if (!label) {return;}
      },
      // 设置选择面板数据
      setSelectorPanelData(result) {
        this.selectorPanelData.optionList = result;
        this.showSelectorPanel = true;
      },
      // input的input事件
      inputEvent(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.targetElementMark = mark;
        this.searchEventName(target.value, this.setSelectorPanelData);
      },
      // input获得焦点
      focus(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.calendarData[mark].canClear = false;
        this.calendarData[mark].border = this.inputHoverBorder;
      },
      // input失去焦点
      blur(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.calendarData[mark].border = this.calendarData[mark].color ? this.inputBorder : this.inputHoverBorder;
      },
      // 清空事件内容
      clearEventContent(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        let eventId = Number(node.getAttribute('data-id'));
        this.calendarData[mark].label = '';
        this.calendarData[mark].color = '';
        this.calendarData[mark].eventId = null;
        this.calendarData[mark].canClear = false;
      },
      mouseover(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.calendarData[mark].border = this.inputHoverBorder;
        if (!this.calendarData[mark].label) {
          for (let k in this.calendarData) {if (k != mark) {this.calendarData[k].placeholder = '';}}
          this.calendarData[mark].placeholder = this.defaultContent;
        } else {
          this.calendarData[mark].canClear = true;
        }
      },
      mouseout(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.calendarData[mark].border = this.calendarData[mark].color ? this.inputBorder : this.inputHoverBorder;
        this.calendarData[mark].placeholder = '';
        this.calendarData[mark].canClear = false;
      },
      // 颜色选择面板按钮点击事件
      colotBtnClick(event) {
        let target = event.target || target.srcElement;
        let node = target.nodeName == 'P' ? target : target.parentNode;
        let mark = node.getAttribute('data-mark');
        this.showColorPanel = false;
        this.targetElementMark = mark;
        setTimeout(()=>{this.showColorPanel = true;},500);
      },
      // 颜色选择面板组件传来的选择的颜色
      colorItemClick({color, targetElementMark}) {
        let event = this.calendarData[targetElementMark];
        let value = event.label;
        if (event.label) {
          if (!event.eventId) {
            this.createCalendarEvent({value, targetElementMark, color, successcb:()=>{
              if (this.showColorPanel) {this.showColorPanel = false;}
            }});
          }
          else {
            this.editCalendarEvent({value, targetElementMark, color, successcb:()=>{
              if (this.showColorPanel) {this.showColorPanel = false;}
            }});
          }
        } else {
          // this.$message({type:'warning',message:'请输入或选择事件名称',duration:2000,center:true});
          this.calendarData[targetElementMark].color = color;
        }
      },
      // 颜色选择面板关闭
      colorPanelColose({targetElementMark}) {
        let event = this.calendarData[targetElementMark];
        let value = event.label;
        let color = event.color;
        if (value && color) {
          if (!event.eventId) {this.createCalendarEvent({value, targetElementMark, color});}
          else {this.editCalendarEvent({value, targetElementMark, color});}
        }
        if (this.showColorPanel) {this.showColorPanel = false;}
      },
      // 选择面板子组件内option的点击事件
      /*
        @ value: 选择面板选中的label值
        @ targetElementMark: 目标元素的标识
        @ targetColor: 目标元素的颜色值
      */
      optionClick({value, targetElementMark, targetColor}) {
        if (!value || !targetElementMark) {return;}
        let event = this.calendarData[targetElementMark];
        let color = targetColor || event.color;
        if (!event.eventId) {this.createCalendarEvent({value, targetElementMark, color});}
        else {this.editCalendarEvent({value, targetElementMark, color});}
      },
      // 选择面板关闭
      selectorPanelColose({targetElementMark}) {
        let event = this.calendarData[targetElementMark];
        let color = event.color, value = event.label, message = '';
        if (value) {
          if (!event.eventId) {this.createCalendarEvent({value, targetElementMark, color});}
          else {this.editCalendarEvent({value, targetElementMark, color});}
        } else {
          // this.$message({type:'warning',message:'请输入或选择事件名称',duration:2000,center:true});
        }
        if (this.showSelectorPanel) {this.showSelectorPanel = false;}
      },
      // 
      getWeek(element) {
        for (let i = 0; i < this.weekNameList.length; i++) {
          if (element.getAttribute('data-week') == this.weekNameList[i]) {return i;}
        }
      },
      // 设置mark标识
      setMark(day, meridiem, index) {
        return day + '--' + meridiem + '--' + (index+1);
      },
      // 设置后的dayList
      takeDayList() {
        let currentYear = this.currentYear || new Date().toJSON().split('-')[0];
        let currentMonth = this.currentMonth || new Date().toJSON().split('-')[1];
        let dayLengthList = this.dayLengthList || [31,28,31,30,31,30,31,31,30,31,30,31];
        let weekNameList = this.weekNameList || ['周日','周一','周二','周三','周四','周五','周六'];
        let weekFirstDayOfMonth = this.weekFirstDayOfMonth || this._witchDayOfTheWeek(new Date().toJSON().split('-')[0], new Date().toJSON().split('-')[1], '01');
        
        if (this._isLeapYear(currentYear)) {dayLengthList.splice(1,1,29);}
        else {dayLengthList.splice(1,1,28);}

        let total = dayLengthList[currentMonth-1] + weekFirstDayOfMonth;
        let rows = Math.ceil(total/weekNameList.length);
        let dayList = [];
        let length = rows * 7;

        for (let i = 0; i < length; i++) {
          if (i < weekFirstDayOfMonth || (i - weekFirstDayOfMonth) >= dayLengthList[currentMonth-1]) {
            dayList.push('');
          } else {
            dayList.push(currentYear+'-'+currentMonth+'-'+((i+1)-weekFirstDayOfMonth<10?'0'+((i+1)-weekFirstDayOfMonth):(i+1)-weekFirstDayOfMonth));
          }
        }
        return dayList;
      },
      // 设置默认的日历数据
      _setDefaultCalendarData() {
        let data = {};
        this.takeDayList().forEach(v=>{
          if (v) {
            data[v+'--0--1'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
            data[v+'--0--2'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
            data[v+'--0--3'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
            data[v+'--1--1'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
            data[v+'--1--2'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
            data[v+'--1--3'] = {eventId:'',label:'',color:'',week:'',eventOrder:'',meridiem:'',placeholder:'',canClear:false,border:'1px dotted #ebeef5'};
          }
        });
        this.calendarData = data;
      },
      // 设置日历数据
      _setCalendarData(data) {
        if (!data || !data.length) {return {};}
        data.forEach(v=>{
          v.eventList.forEach(y=>{
            let obj = {};
            obj = {
              eventId:y.eventId, label:y.label, color:y.color, week:y.week, eventOrder:y.eventOrder, meridiem:y.meridiem,
              placeholder:'', canClear:false, border:this.inputBorder
            }
            this.calendarData[v.eventDate+'--'+y.meridiem+'--'+y.eventOrder] = obj;
          });
        });
      },
      // 获取组件data属性内的指定日期指定order的calendar数据
      getDayEvent(mark) {
        if (!mark) {return {};}
        return this.calendarData[mark];
      },
      // 返回day
      /*
        @ day: 日期 2019-01-01
        @ type: 返回的数据的类型 Number|String
      */
      returnDay(day, type='String') {
        if (!day || !type) {return '';}
        switch (type) {
          case 'Number': return Number(day.split('-')[2]); break;
          case 'String': return day.split('-')[2]; break;
        }
      },
      // 渲染日期
      renderDay(rowIndex, cloumIndex) {
        if (rowIndex == 0) {
          if (cloumIndex < this.weekFirstDayOfMonth) {return '';}
          else {
            if (cloumIndex == this.weekFirstDayOfMonth) {return 1;}
            else {return 1+cloumIndex-this.weekFirstDayOfMonth;}
          }
        }
        else {
          let day = rowIndex * 7 - this.weekFirstDayOfMonth + cloumIndex + 1;
          return day <= this.dayLengthList[Number(this.currentMonth)-1] ? day : '';
        }
      },
      // 前一月
      previousMonth() {
        let m = Number(this.currentMonth);
        if (m === 1) {
          this.currentYear = (Number(this.currentYear) - 1) + '';
          this.currentMonth = '12';
        }
        else {
          if (m <= 10) {this.currentMonth = '0'+ (m-1);}
          else {this.currentMonth = ''+(m-1);}
        }
        this.weekFirstDayOfMonth = this._witchDayOfTheWeek(this.currentYear, this.currentMonth, '01');
        this.getCalendarList(this._setCalendarData);
      },
      // 后一月
      nextMonth() {
        let m = Number(this.currentMonth);
        if (m === 12) {
          this.currentYear = (Number(this.currentYear) + 1) + '';
          this.currentMonth = '01';
        }
        else {
          if (m < 9) {this.currentMonth = '0'+ (m+1);}
          else {this.currentMonth = ''+(m+1);}
        }
        this.weekFirstDayOfMonth = this._witchDayOfTheWeek(this.currentYear, this.currentMonth, '01');
        this.getCalendarList(this._setCalendarData);
      },
      // 设置colum的class
      makeColumClass(day, i) {
        let className = '';
        if (i % 7 == 0) {
          className = 'noLeftBorder bg-gray';
        }
        if (i % 7 == 6) {
          className = 'borderRight bg-gray';
        }
        return className;
      },
      // 时间戳转时间
      _timestampToTime(timestamp) {
        let date = new Date();
        date.setTime(timestamp);
        let dateList = date.toJSON().split('T')[0].split('-');

        return dateList[0] + '年' + dateList[1] + '月';
      },
      // 是否是闰年
      _isLeapYear(year) {
        if (!year) {throw error('_isLeapYear方法参数错误'); return;}
        if ((year % 4 === 0) && (year % 100 !== 0)) {return true;}
        if ((year % 4 === 0) && (year % 100 === 0) && (year % 400 === 0)) {return true;}
        return false;
      },
      // 是星期几
      _witchDayOfTheWeek(year, month, day) {
        let T = new Date(year + '-' + month + '-' +day);
        return T.getDay();
      },
      // 根据年月、团队查询日历
      getCalendarList(callback) {
        
      },
      // 创建日历事件
      createCalendarEvent({value, targetElementMark, color, successcb, errorcb}) {
        let element = document.querySelector('[data-mark="'+targetElementMark+'"]');
        let params = {
          eventDate: element.getAttribute('data-mark').split('--')[0],
          meridiem: element.getAttribute('data-mark').split('--')[1],
          eventOrder: element.getAttribute('data-mark').split('--')[2],
          week: this.getWeek(element)
        }
        
      },
      // 编辑日历事件
      editCalendarEvent({value, targetElementMark, color, successcb, errorcb}) {
        let element = document.querySelector('[data-mark="'+targetElementMark+'"]');
        let params = {eventId: element.getAttribute('data-id')}
        
      },
    },
    destroyed(){},
  }
</script>
<style>

</style>
<style lang="stylus" rel="stylesheet/stylus" scoped>
  .calendar
    width 100%
    position relative
    min-width 1112px
    margin-bottom 30px
    border 1px solid transparent
    box-sizing border-box

    .calendar-container
      margin-top 10px
      width 100%
      min-width 1112px
      background-color #fff
      -webkit-border-radius 5px
      -moz-border-radius 5px
      -ms-border-radius 5px
      border-radius 5px

      .calendar-title
        height 30px
        line-height 30px
        text-align center
        font-size 14px
        border-bottom 1px solid #eee
        user-select none
        .year-month
          padding 0 15px
          color #333
          font-size 12px
        .el-icon-circle-close
          cursor pointer
          font-size 10px
          padding 0 3px
          color #333

      .calendar-week
        display flex
        border-bottom 1px solid #00cccc
        height 30px
        .week-name
          //flex 1
          //float left
          //width 158px
          width 14%
          font-size 12px
          text-align center
          font-weight 500
          padding 9px 0px
          color #333
          border-left 1px solid #ebeef5
          user-select none
        .week-name:first-child
          border-left 0px
        .week-name:last-child
          border-right 1px solid #ebeef5

      .calendar-day-container
        position relative
        display flex
        flex-wrap wrap
        .calendar-day
          border-left 1px solid #ebeef5
          border-bottom 1px solid #94e1e1
          background-color #fff
          position relative
          //flex-grow 0
          //min-width 158px
          //flex 1
          width 14%
          span.day
            position absolute
            right 0
            top 0
            padding 5px 3px
            font-size 12px
            color #333
          .am-box,.pm-box
            width 100%
            height 80px
            position relative
          .am-box
            border-bottom 1px dotted #ebeef5
            display flex
            .noon-box
              width 14px
              height 80px
              flex 1
              min-width 14px
              max-width 14px
              position relative
              .am-span
                position absolute
                bottom 3px
                left 0px
                width 14px
                font-size 10px
                color #666
                line-height 18px
                margin 0 auto 
                padding 10px 0px
                background-color #ebeef5
            .noon-box + .am-input-box
              width calc(100% - 18px)
            .am-input-box
              flex 5
              width calc(100% - 4px)
              padding 0px 3px 0px 1px
              height 56px
              position relative
              right 0px
              bottom -20px
          .pm-box
            display flex
            .noon-box
              width 14px
              height 80px
              flex 1
              min-width 14px
              max-width 14px
              position relative
              .pm-span
                position absolute
                top 3px
                left 0px
                width 14px
                font-size 10px
                color #666
                line-height 18px
                margin 0 auto 
                padding 10px 0px
                background-color #ebeef5
            .noon-box + .pm-input-box
              width calc(100% - 18px)
            .pm-input-box
              flex 5
              width calc(100% - 4px)
              padding 0px 3px 0px 1px
              height 56px
              position relative
              right 0px
              top 3px
        .borderRight
          border-right 1px dotted #ebeef5
        .noLeftBorder
          border-left 0px
          

        .event-input
          margin 0px
          display flex
          height 17.5px
          line-height 17.5px
          padding 0px 3px
          font-size 10px
          color #666
          border 1px dotted transparent
          position relative
          .event-content
            position absolute
            width 100%
            cursor pointer
            text-align center
          input
            flex 6
            height 100%
            background none
            border none
            overflow hidden
            text-overflow ellipsis
            white-space nowrap
            &:focus
              outline inherit
          ::-webkit-input-placeholder
            color #8fcfff
            text-align center
          :-moz-placeholder
            color #8fcfff
            text-align center
          ::-moz-placeholder
            color #8fcfff
            text-align center
          :-ms-input-placeholder
            color #8fcfff
            text-align center
          i
            font-size 12px
            cursor pointer
            margin-right 2px
            position absolute
            right 15px
            top 3px
          i.white
            color #fff
          i.gray
            color #999
          em
            flex 1
            max-width 8px
            //display none
            width 8px
            height 8px
            position relative
            top 50%
            right 0px
            cursor pointer
            border 1px solid #ebeef5
            -webkit-transform translateY(-50%)
            -moz-transform translateY(-50%)
            -ms-transform translateY(-50%)
            transform translateY(-50%)
            -webkit-border-radius 2px
            -moz-border-radius 2px
            -ms-border-radius 2px
            border-radius 2px
        

  .bg-gray
    background-color #f7f7f7!important
    border-bottom none

</style>
