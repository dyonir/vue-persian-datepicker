<template>
    <div class="pdatepicker" v-bind:class="[ {'inline' : inlineMode}, wrapperClass ]">
        <input :id="id"
            type="text"
               @click="inputClicked"
               :value="formatedChosenDate"
               @change="inputChanged($event.target.value)"
               :class="inputClass"
               :name="name"
               :placeholder="placeholder"
               :readonly="inputDisabled">
        <transition name='fade' v-if='modalMode && isDialogOpen'>
            <div class="modal-overlay"
                v-if='isDialogOpen'
                @click='isDialogOpen = false'>
            </div>
        </transition>
        <transition :name="modalMode ? modalOpenTransitionAnimation : openTransitionAnimation">
            <div
                v-if="isDialogOpen"
                v-on:click.stop
                v-bind:class="[{ 'modal-dialog' : modalMode }, dialogClass , 'pdatepicker-dialog']"
                v-bind:style="{ dialogDynamicStyle }">
                <transition name="fade">
                    <div class='day-view' v-if='isDayView'>
                        <div class="pdatepicker-dialog-header" v-bind:style='{background : headerBackgroundColor, color: headerColor}'>
                            <div class='pdatepicker-dialog-month'>
                                <div class="preMonth" @click='preMonthClicked'>&lt;</div>
                                <div class="monthName"@click='goToMonthSelect'>{{ displayingMonth }} {{ numToStr(displayingYear) }}</div>
                                <div class="nextMonth" @click='nextMonthClicked'>&gt;</div>
                            </div>
                        </div>
                        <div class='pdatepicker-dialog-days'>
                            <div class='day-box day-name' v-for='dayName in dayNames'>
                                <span>
                                {{ dayName }}
                                </span>
                            </div>
                            <div class='day-box empty-box' v-for='n in firstDayOfMonth'></div>
                            <template v-for='n in daysInMonth'>
                                <div class='day-box'
                                    v-bind:class="{'disabled-day' : !isDateInRange(n) , 'chosen-day' :  ifDayBoxIsChosenDay(n) }"
                                    @click='dayClicked(n)'>
                                    <span class="hover-effect"
                                    v-bind:style="{ 'background-color': !ifDayBoxIsChosenDay(n) ? hoverDayBackColor : chosenDayBackColor  }"></span>
                                    <span class='num'>
                                        {{ numToStr(n) }}
                                    </span>
                                </div>
                            </template>
                        </div>
                    </div>
                </transition>
                <transition name="fade">
                    <div class='month-view' v-if='isMonthView'>
                        <div class="pdatepicker-dialog-header" v-bind:style='{background : headerBackgroundColor, color: headerColor}'>
                            <div class='pdatepicker-dialog-year'>
                                <div class="preYear" @click='preYearClicked'>&lt;</div>
                                <div class="cyear" @click="goToYearSelect">{{ numToStr(displayingYear) }}</div>
                                <div class="nextYear" @click='nextYearClicked'>&gt;</div>
                            </div>
                        </div>
                        <div class='pdatepicker-dialog-months'>
                            <template v-for='(n, i) in monthNames'>
                                <div class='month-box'
                                    v-bind:class="{ 'chosen-month' : ifMonthBoxChosenMonth(i) }"
                                    @click='monthClicked(i)'>
                                    <span class="hover-effect"
                                    v-bind:style="{ 'background-color': !ifMonthBoxChosenMonth(n) ? hoverDayBackColor : chosenDayBackColor  }"></span>
                                    <span class="num">
                                    {{ n }}
                                    </span>
                                </div>
                            </template>
                        </div>
                    </div>
                </transition>
                <transition name="fade">
                    <div class='year-view'
                        v-if='isYearView'>
                        <template v-for='n in (cMaximumYear - cMinimumYear + 1)'>
                            <div class='year-box'
                                :id="'year-' + (n + cMinimumYear - 1)"
                                v-bind:class="{ 'chosen-year' : ifYearBoxChosenYear(n + cMinimumYear - 1) }"
                                @click="yearClicked(n + cMinimumYear - 1)">
                                <span class="hover-effect"
                                    v-bind:style="{ 'background-color': !ifYearBoxChosenYear(n + cMinimumYear - 1) ? hoverDayBackColor : chosenDayBackColor  }">
                                </span>
                                <span class="num">
                                {{ numToStr(n + cMinimumYear - 1) }}
                                </span>
                            </div>
                        </template>
                    </div>
                </transition>
            </div>
        </transition>
    </div>
</template>

<script>
export default {
  name : 'PDatePicker',
  props : {
      'placeholder' : { default : 'یک تاریخ را انتخاب کنید', type: String},
      'headerBackgroundColor' :{ default : '#137e95' , type: String },
      'headerColor' : { default : 'white', type: String},
      'dialogColor' : { default : 'black', type: String},
      'dialogBackgroundColor' : { default : '#fafafa', type: String},
      'hoverDayBackColor' : { default : '#1af7ff', type: String },
      'chosenDayBackColor' : { default : '#1ad7ff', type: String },
      'minimumYear' : { default : 1300, type: Number},
      'maximumYear' : { default : 1450, type: Number },
      'disableDatesBeforeToday' : { default : false, type: Boolean },
      'availableDates' : { default : false, type: Boolean },
      'availableDateStart' : { default: '1300/01/01', type: String,
          validator (value){
              if(value === '') return true;
              let elements = value.split('/');
              if(elements.length !== 3) return false;
              if(parseInt(elements[0]) < 1300) return false;
              let month = parseInt(elements[1]);
              if(month < 1 || month > 12) return false;
              let day = parseInt(elements[2]);
              if(day < 1 || day > 31) return false;
              return true;
          }
      },
      'availableDateEnd' : { default: '1450/12/29', type: String,
          validator (value){
              if(value === '') return true;
              let elements = value.split('/');
              if(elements.length !== 3) return false;
              if(parseInt(elements[0]) < 1300) return false;
              let month = parseInt(elements[1]);
              if(month < 1 || month > 12) return false;
              let day = parseInt(elements[2]);
              if(day < 1 || day > 31) return false;
              return true;
          }
      },
      'value' : { default : '' },
      'name' : { default : '', type : String },
      'required' : { default : false,type : Boolean },
      'id' : { default : '', type : String},
      'inputClass': { default : '', type :String },
      'dialogClass' :  {default : '',type : String },
      'wrapperClass' :  {default : '',type : String },
      'initialView' : { default: 'day',type : String,
              validator (value){
                  return value === 'day' || value === 'month'
              }
          },
      'inlineMode' : { default : false, type :Boolean },
      'inputDisabled' : { default : true, type :Boolean },
      'formatDate' : { default: 'yyyy/MM/dd',type : String,
              validator (value) {
                  let elements = value.split("/");
                  if(elements.length !== 3) return false;
                  if(elements[0] !== "yyyy" && elements[0] !== "yy") return false;
                  if(elements[1] !== "M" && elements[1] !== "MM" && elements[1] !== "MMM") return false;
                  if(elements[2] !== "d" && elements[2] !== "dd") return false;
                  return true;
              }
          },
      'openTransitionAnimation' : { default: 'slide-fade' ,type : String },
      'persianDigits' : { default : true,type : Boolean },
      'modalMode' : { default: false,type : Boolean },
      'modalOpenTransitionAnimation' : { default: 'scale-fade', type : String }
  },
  data () {
    return {
        isDialogOpen : false,
        isDayView : true,
        isMonthView : false,
        isYearView: false,
        dayNames : ['ش', 'ی', 'د', 'س', 'چ', 'پ', 'ج'],
        monthNames : ['فروردین', 'اردیبهشت', 'خرداد', 'تیر', 'مرداد', 'شهریور' ,'مهر' ,'آبان' ,'آذر', 'دی' ,'بهمن', 'اسفند'],
        dayInThisMonth: 1,
        firstDayOfMonth: 0,
        daysInMonth: 30,
        gtoday : [1380, 1, 1],
        displayingMonthNum : 1,
        displayingMonth : '',
        displayingYear : 1300,
        dayOfWeek: 0,
        chosenDate : '',
        formatedChosenDate : '',
        chosenDay: 1,
        chosenMonth : 1,
        chosenYear : 1396,
        startAvailableDateV : {
            year : 1300,
            month: 1,
            day : 1
        },
        endAvailableDateV : {
            year : 1450,
            month: 12,
            day : 29
        }
    }
  },
  computed:{
        dialogDynamicStyle(){
            return {
                background: this.dialogBackgroundColor,
                color: this.dialogColor
            }
        },
        chosenDayDynamicStyle(){
            return {
                background: this.chosenDayColor
            }
        },
        cMinimumYear(){
            if(this.startAvailableDateV.year > this.minimumYear) {
                return this.startAvailableDateV.year;
            }
            return this.minimumYear;
        },
        cMaximumYear(){
            if(this.endAvailableDateV.year < this.maximumYear) {
                return this.endAvailableDateV.year;
            }
            return this.maximumYear;
        }
  },
  mounted(){
    if(this.availableDates){

        let elements = this.availableDateStart.split("/");
        this.startAvailableDateV.year = parseInt(elements[0]);
        this.startAvailableDateV.month = parseInt(elements[1]);
        this.startAvailableDateV.day = parseInt(elements[2]);

        elements = this.availableDateEnd.split("/");
        this.endAvailableDateV.year = parseInt(elements[0]);
        this.endAvailableDateV.month = parseInt(elements[1]);
        this.endAvailableDateV.day = parseInt(elements[2]);

    }

    if(this.disableDatesBeforeToday){
        this.availableDates = true;
        let today = new Date();
        let gToday = this.gregorian_to_jalali(today.getFullYear(), today.getMonth() + 1, today.getDate());
        let gTodayNum = gToday[0] * 10000 +
                gToday[1] * 100 +
                gToday[2];

        let sdate = this.startAvailableDateV.year * 10000 +
                (this.startAvailableDateV.month) * 100 +
                this.startAvailableDateV.day;

        if(sdate - gTodayNum < 0){
            this.startAvailableDateV.year = gToday[0];
            this.startAvailableDateV.month = gToday[1];
            this.startAvailableDateV.day = gToday[2];
        }
    }




    if(this.inputCheck(this.value)){
        this.inputChanged(this.value);
    } else if(this.isToDayInRange()){
        this.goToToday();
    } else {
        this.goToMonth(this.startAvailableDateV.year , this.startAvailableDateV.month - 1, this.startAvailableDateV.day);
    }

    if(this.inlineMode){
        this.openDialog();
    }

    if (!this.inlineMode && !this.modalMode){
        document.documentElement.addEventListener('click', this.onExit, false);
    }
  },
  beforeDestroy: function () {
    if (!this.inlineMode && !this.modalMode){
        document.documentElement.removeEventListener('click', this.onExit, false);
    }
  },
  watch:{
      value : function(value){
          this.inputChanged(value);
      }
  },
  methods: {
    inputClicked () {
        if(!this.isDialogOpen)
            this.openDialog();
        else
            this.closeDialog();
    },
    hasInputClass(){
        return inputClass !== '';
    },
    openDialog(){
        if(this.isDialogOpen) return;
        this.isDialogOpen = true;
        if(this.initialView === 'day'){
            this.isDayView = true;
            this.isMonthView = false;
            this.isYearView = false;
        } else if(this.initialView === 'month') {
            this.isDayView = false;
            this.isMonthView = true;
            this.isYearView = false;
        } else {
            this.isDayView = false;
            this.isMonthView = false;
            this.isYearView = true;
        }
        this.$emit('opened', this.value);
    },
    closeDialog(){
        if(!this.inlineMode){
            this.isDialogOpen = false;
            this.isDayView = false;
            this.isMonthView = false;
            this.isYearView = false;
            this.$emit('closed', this.value);
        }
    },
    isDateInRange(day){
        if(!this.availableDates) return true;
        let cdate = this.displayingYear * 10000 +
                (this.displayingMonthNum + 1) * 100 +
                day;

        let sdate = this.startAvailableDateV.year * 10000 +
                (this.startAvailableDateV.month) * 100 +
                this.startAvailableDateV.day;

        let edate = this.endAvailableDateV.year * 10000 +
                (this.endAvailableDateV.month ) * 100 +
                this.endAvailableDateV.day;
       return (cdate - sdate >= 0) && (cdate - edate <= 0);
    },
    isToDayInRange(){
        if(!this.availableDates) return true;
        let today = new Date();
        let gtoday = this.gregorian_to_jalali(today.getFullYear(), today.getMonth() + 1, today.getDate());
        let cdate = gtoday[0] * 10000 + (gtoday[1]) * 100 + gtoday[2];

        let sdate = this.startAvailableDateV.year * 10000 +
                (this.startAvailableDateV.month) * 100 +
                this.startAvailableDateV.day;

        let edate = this.endAvailableDateV.year * 10000 +
                (this.endAvailableDateV.month ) * 100 +
                this.endAvailableDateV.day;
       return (cdate - sdate >= 0) && (cdate - edate <= 0);
    },
    inputCheck(value){
        if(value !== ''){
            let els = value.split("/");
            if(els.length === 3){
                let year = parseInt(this.convertDigitsPTE(els[0]));
                let month = parseInt(this.convertDigitsPTE(els[1]));
                let day = parseInt(this.convertDigitsPTE(els[2]));
                if(isNaN(month)){
                    month = this.monthNames.indexOf(els[1]) + 1;
                }
                if(!isNaN(year) && !isNaN(day) && month !== -1){
                    if(month < 1 || month > 12) return false;
                    if(month <= 6 && (day < 1 || day > 31)) return false;
                    if(month > 6 && (day < 1 || day > 30)) return false;
                    if(year < 1300) year += 1300;
                    if(year < this.cMinimumYear || year > this.cMaximumYear) return false;
                    return true;
                }
            }
        }
        return false;
    },
    inputChanged(value){
        if(this.inputCheck(value)) {
            let els = value.split("/");
            let year = parseInt(this.convertDigitsPTE(els[0]));
            let month = parseInt(this.convertDigitsPTE(els[1]));
            let day = parseInt(this.convertDigitsPTE(els[2]));
            if(isNaN(month)){
                month = this.monthNames.indexOf(els[1]) + 1;
            }
            this.goToMonth(year, month - 1, day);
            // this.updateInput();
            // this.dayClicked(day);
        }
    },
    ifDayBoxIsChosenDay(day){
        return this.chosenYear === this.displayingYear &&
                this.chosenMonth === (this.displayingMonthNum + 1) &&
                this.chosenDay === day;
    },
    ifMonthBoxChosenMonth(month){
        return this.chosenYear === this.displayingYear &&
                this.chosenMonth === month + 1;
    },
    ifYearBoxChosenYear(year){
        return this.chosenYear === year;
    },
    goToToday(){
        let today = new Date();
        this.gtoday = this.gregorian_to_jalali(today.getFullYear(), today.getMonth() + 1, today.getDate());
        this.chosenDay = this.gtoday[2];
        this.chosenMonth = this.gtoday[1];
        this.chosenYear = this.gtoday[0];
        
        this.goToMonth(this.chosenYear, this.chosenMonth - 1, this.chosenDay);
    },
    goToMonth(year, month, day){
        var gfirstOfMonth = this.jalali_to_gregorian(year, month + 1, 1);

        var firstOfMonth = new Date(gfirstOfMonth[0], gfirstOfMonth[1] - 1, gfirstOfMonth[2] + 1);
        this.firstDayOfMonth = (firstOfMonth.getDay()) %7 ;
        this.daysInMonth = this.gatDaysInMonth(month);
        this.displayingMonthNum = month;

        let yearch = this.displayingYear !== year;
        let monthch = this.displayingMonth !== this.monthNames[month];

        this.displayingYear = year;
        this.displayingMonth = this.monthNames[month];

        if(monthch)
            this.$emit('monthChanged', this.value);
        if(yearch)
            this.$emit('yearChanged', this.value);
    },
    gatDaysInMonth(monthNumber){
        if(monthNumber == 11){
            return this.isLeapYear(this.displayingYear) ? 30 : 29;
        }
        return monthNumber <= 5 ? 31 : 30;
    },
    isLeapYear(year){
        let rm = year % 33;
        if(year <= 1342)
            return rm == 1 || rm == 5 || rm == 9 || rm == 13 || rm == 17 || rm == 21 || rm == 26 || rm == 30;
        return rm == 1 || rm == 5 || rm == 9 || rm == 13 || rm == 17 || rm == 22 || rm == 26 || rm == 30;
    },
    preMonthClicked(){
        let newMonth = this.displayingMonthNum - 1;
        let newYear = this.displayingYear;
        if(newMonth < 0){
            if(newYear - 1 >= this.cMinimumYear) {
                newMonth = 11;
                newYear--;
            } else {
                newMonth = 0;
            }
        }
        this.goToMonth(newYear, newMonth, 1);
    },
    nextMonthClicked(){
        let newMonth = this.displayingMonthNum + 1;
        let newYear = this.displayingYear;
        if(newMonth > 11){
            if(newYear + 1 <= this.cMaximumYear) {
                newMonth = 0;
                newYear++;
            } else {
                newMonth = 11;
            }
        }
        this.goToMonth(newYear, newMonth, 1);
    },
    dayClicked(day){
        if(!this.isDialogOpen ||
            (this.availableDates && !this.isDateInRange(day))){
            return;
        }
        this.chosenDay = day;
        this.chosenMonth = this.displayingMonthNum + 1;
        this.chosenYear = this.displayingYear;
        this.updateInput();
        this.closeDialog();
    },
    monthClicked(month){
        this.displayingMonthNum = month;
        this.isMonthView = false;
        this.isYearView = false;
        this.isDayView = true;
        this.goToMonth(this.displayingYear, this.displayingMonthNum, 1);
    },
    yearClicked(year){
        this.displayingYear = year;
        this.isMonthView = true;
        this.isYearView = false;
        this.isDayView = false;
        this.goToMonthSelect();
    },
    goToYearSelect(event){
        this.isMonthView = false;
        this.isDayView = false;
        this.isYearView = true;
        this.$nextTick(function () {
            let target = this.$el.querySelector('#year-' + this.displayingYear);
            target.parentNode.scrollTop = target.offsetTop - target.parentNode.offsetTop;
        });
    },
    updateInput(){
        let str = this.toFormatDate(this.chosenYear, this.chosenMonth, this.chosenDay);
        if(this.persianDigits)
            this.formatedChosenDate = this.convertDigitsETP(str);
        else
            this.formatedChosenDate = this.convertDigitsPTE(str);
        this.$emit('selected', { year: this.chosenYear, month: this.chosenMonth, day: this.chosenDay});
        this.$emit('input', str);
    },
    numToStr(num){
        if(this.persianDigits){
            return this.convertDigitsETP('' + num);
        }
        return '' + num;
    },
    nextYearClicked(){
        if(this.displayingYear + 1 <= this.cMaximumYear) {
            this.displayingYear++;
            this.$emit('yearChanged', this.value);
        }
    },
    preYearClicked(){
        if(this.displayingYear - 1 >= this.cMinimumYear) {
            this.displayingYear--;
            this.$emit('yearChanged', this.value);
        }
    },
    goToMonthSelect(){
        this.isDayView = false;
        this.isMonthView = true;
        this.chosenMonth = this.displayingMonthNum + 1;
        this.$emit('monthChanged', this.value);
    },
    toFormatDate(year, month, day){
        let elements = this.formatDate.split("/");
        let outYear = '' + elements[0] === "yyyy" ? year : year - 1300;
        let outMonth = '';
        if(elements[1] === 'M') outMonth = month;
        else if(elements[1] === 'MM'){
            if(month < 10) outMonth = '0' + month;
            else outMonth =  month;
        }
        else if(elements[1] === 'MMM') outMonth = this.monthNames[month - 1];
        let outDay = elements[2] === 'dd' && day < 10 ? '0' + day : day;
        return outYear + "/" + outMonth + "/" + outDay;
    },
    onExit(ev) {
        if (!this.$el.contains(ev.target))
            this.closeDialog();
    },    
    /**
     * This function convert english digits to persian ones.
     * @param {String} unconverted string
     * @returns {String} converted string
     */
    convertDigitsPTE(str){
      return this.replaceAllArray(str,
        ['۰', '۱', '۲', '۳', '۴', '۵', '۶', '۷', '۸', '۹'],
        ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']);
    },
    /**
     * This function convert persian digits to english ones.
     * @param {String} unconverted string
     * @returns {String} converted string
     */
    convertDigitsETP(str){
      return this.replaceAllArray(str,
        ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
        ['۰', '۱', '۲', '۳', '۴', '۵', '۶', '۷', '۸', '۹']);
    },
    replaceAllArray(str, find, replace){
        let replacedStr = str;
        for(let i=0; i<find.length; i++){
            replacedStr = replacedStr.split(find[i]).join(replace[i]);
        }
        return replacedStr;
    },
    gregorian_to_jalali( gy, gm, gd) {
        var g_d_m=[0,31,59,90,120,151,181,212,243,273,304,334];
        var jy, jm, jd, gy2, days;
        if(gy > 1600) {
            jy=979;
            gy-=1600;
        } else {
            jy=0;
            gy-=621;
        }
        gy2=(gm > 2)?(gy+1):gy;
        days=(365*gy) +(parseInt(( gy2 + 3 ) / 4)) -(parseInt((gy2+99)/100)) +(parseInt((gy2+399)/400)) -80 +gd +g_d_m[gm-1];
        jy += 33 * (parseInt( days / 12053));
        days%=12053;
        jy += 4*(parseInt(days/1461));
        days%=1461;
        if(days > 365){
            jy+=parseInt((days-1)/365);
            days=(days-1)%365;
        }
        jm=(days < 186)?1+parseInt(days/31):7+parseInt((days-186)/30);
        jd=1+((days < 186)?(days%31):((days-186)%30));
        return [jy,jm,jd];
    },
    jalali_to_gregorian( jy, jm, jd) {
        var gy, gm, gd, days, sal_a, v;
        if(jy > 979){
            gy=1600;
            jy-=979;
        }else{
            gy=621;
        }
        days=(365*jy) +((parseInt(jy/33))*8) +(parseInt(((jy%33)+3)/4)) +78 +jd +((jm<7)?(jm-1)*31:((jm-7)*30)+186);
        gy+=400*(parseInt(days/146097));
        days%=146097;
        if(days > 36524) {
            gy+=100*(parseInt(--days/36524));
            days%=36524;
            if(days >= 365)days++;
        }
        gy+=4*(parseInt(days/1461));
        days%=1461;
        if(days > 365){
            gy+=parseInt((days-1)/365);
            days=(days-1)%365;
        }
        gd=days+1;
        sal_a=[0,31,((gy%4==0 && gy%100!=0) || (gy%400==0))?29:28,31,30,31,30,31,31,30,31,30,31];
        for(gm=0; gm < 13; gm++) {
            v=sal_a[gm];
            if(gd <= v)break;
            gd-=v;
        }
        return [gy,gm,gd];
    }
  }
}
</script>

<style lang='scss' scoped>
    @mixin clearfix() {
        &::after {
          display: block;
          content: "";
          clear: both;
        }
    }
    $dialog-width: 270px;
    $box-width : $dialog-width / 7;
    $month_box_width : $dialog-width / 3;
    $font-size: 14px;

    .pdatepicker{
        position: relative;
        display: inline-block;
        * {
            box-sizing: border-box;
        }
        input{
            text-align: left;
            direction: rtl;
            width : $dialog-width;
            background-color: white;
            color: black;
        }
        .pdatepicker-dialog{
            position: absolute;
            border: 1px solid gray;
            box-shadow: 0px 0px 2px 0px gray;
            background-color: #fafafa;
            z-index: 100000;
            width: $dialog-width + 2;
            font-size: 0;

            span{
                font-size: $font-size;
            }
            .pdatepicker-dialog-header{
                width: 100%;
                box-shadow: 0px 0px 5px 0px gray;
                @include clearfix;
            }
            .endofweek, .endofseason{
                padding: 0px;
                margin: 0px;
                width: 0px;
                height: 0px;
                @include clearfix();
            }
            .day-view{
                text-align: right;
                .pdatepicker-dialog-month{
                    width: 100%;
                    div{
                        display: inline-block;
                        text-align: center;
                        padding: 10px 0;
                        cursor: pointer;
                        font-size: $font-size;
                    }
                    .nextMonth{
                        float: right;
                        width: 10%;
                        &:hover{
                            background-color: rgba(200, 200, 200, 0.6);
                        }
                    }
                    .preMonth{
                        float: right;
                        width: 10%;
                        &:hover{
                            background-color: rgba(200, 200, 200, 0.6);
                        }
                    }
                    .monthName{
                        float: right;
                        width: 80%;
                        &:hover{
                            background-color: rgba(200, 200, 200, 0.6);
                        }
                    }
                }
                .pdatepicker-dialog-week{
                    width: 100%;
                }
                .day-box{
                    width: $box-width;
                    height: $box-width;
                    line-height: $box-width;
                    display: inline-block;
                    text-align: center;
                    border: 1px solid transparent;
                    padding:0;
                    cursor: pointer;
                    vertical-align: middle;
                    border-radius: 50%;
                    position: relative;
                    .hover-effect{
                        position: absolute;
                        top: 0px;
                        right: 0px;
                        width: 100%;
                        height: 100%;
                        border-radius: 50%;
                        transition: transform 150ms ease-out;
                        z-index: 1;
                        transform: scale(0);
                    }
                    .num{
                        z-index: 2;
                        position: relative;
                    }
                    &:hover{
                        border: 1px solid transparent;
                        .hover-effect{
                            transform: scale(1);
                        }
                    }
                    &.disabled-day{
                        background-color: #e3e3e3;
                        cursor: default;
                        .hover-effect{
                            display: none !important;
                        }
                    }

                    &.chosen-day{
                        .hover-effect{
                            transform: scale(1) !important;
                        }
                    }


                }

                .day-name{
                    border-bottom: 1px solid gray;
                    &:hover{
                        cursor: default;
                        border-bottom: 1px solid gray;
                    }
                }
                .empty-box{
                    cursor: default;
                    &:hover{
                        border: inherit;
                    }
                }
            }
            .month-view{
                text-align: center;
                .pdatepicker-dialog-year{
                    width: 100%;
                    div{
                        display: inline-block;
                        text-align: center;
                        padding: 10px 0;
                        cursor: pointer;
                        font-size: $font-size;
                    }
                    .nextYear{
                        float: right;
                        width: 10%;
                    }
                    .preYear{
                        float: right;
                        width: 10%;
                    }
                    .cyear{
                        float: right;
                        width: 80%;
                    }
                }
                .month-box{
                    width: $month_box_width;
                    line-height: $box-width;
                    display: inline-block;
                    text-align: center;
                    border: 1px solid rgba(200, 200, 200, 0);
                    padding:0 5px;
                    cursor: pointer;
                    font-size: $font-size;
                    position: relative;
                    .hover-effect{
                        position: absolute;
                        top: 0px;
                        right: 0px;
                        width: 100%;
                        height: 100%;
                        transition: transform 150ms ease-out;
                        z-index: 1;
                        transform: scale(0);
                        z-index: 1;
                    }
                    .num{
                        position: relative;
                        z-index: 2;
                    }
                    &:hover{
                        border: 1px solid rgb(200, 200, 200);
                        .hover-effect{
                            transform: scale(1);
                        }
                    }
                    &.chosen-month{
                        .hover-effect{
                            transform: scale(1) !important;
                        }
                    }
                }
            }
            .year-view{
                width: 100%;
                height: 250px;
                overflow: scroll;
                overflow-x: hidden;
                .year-box{
                    display: inline-block;
                    text-align: center;
                    padding: 10px 0;
                    cursor: pointer;
                    font-size: $font-size;
                    width: 25%;
                    position: relative;
                    border: 1px solid rgba(200, 200, 200, 0);
                    .hover-effect{
                        position: absolute;
                        top: 0px;
                        right: 0px;
                        width: 100%;
                        height: 100%;
                        transition: transform 150ms ease-out;
                        z-index: 1;
                        transform: scale(0);
                        z-index: 1;
                    }
                    .num{
                        position: relative;
                        z-index: 2;
                    }
                    &:hover{
                        border: 1px solid rgb(200, 200, 200);
                        .hover-effect{
                            transform: scale(1);
                        }
                    }
                    &.chosen-year{
                        .hover-effect{
                            transform: scale(1) !important;
                        }
                    }
                }
            }
        }
        &.inline{
            display: inline-block;
            .pdatepicker-dialog{
                 position: static;
            }
        }
    }

    .modal-overlay{
        position: fixed;
        top: 0px;
        right: 0px;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.6);
        z-index: 10000;
    }
    .modal-dialog{
        position: fixed !important ;
        right: 50%;
        top: 50%;
        transform: translate(50%, -50%);
        z-index: 10001;
        box-shadow: 0px 0px 5px 2px rgb(26, 26, 26);
        transition: all 150ms ease-out;
    }

    /*** Animations ***/
    /** Scale Fade **/
    .fade-enter-active {
        transition: all 300ms ease-out;
    }
    .fade-leave-active {
        transition: all 300ms ease-out;
    }

    .fade-leave-to, .fade-enter
    {
        opacity: 0;
        position: absolute;
        top: 0px;
    }

    .fade-enter-to, .fade-leave{
        opacity: 1;
    }

    /** Scale Fade **/
    .scale-fade-enter-active {
        transition: all 300ms ease-out;
    }
    .scale-fade-leave-active {
        transition: all 300ms ease-out;
    }

    .scale-fade-enter, .scale-fade-leave-to
    {
        transform: translate(50%, -50%) scale(0.7);
        opacity: 0;
    }

    .scale-fade-enter-to{
        transform: translate(50%, -50%) scale(1.05);
        opacity: 1;
    }
    .scale-fade-leave{
        transform: translate(50%, -50%) scale(1.1);
        opacity: 1;
    }

    /** Slide Fade **/
    .slide-fade-enter-active {
        transition: all 300ms ease-out;
    }
    .slide-fade-leave-active {
        transition: all 300ms ease-out;
    }

    .slide-fade-enter, .slide-fade-leave-to
    {
        transform: translateY(-10px);
        opacity: 0;
    }

    .slide-fade-enter-to, .slide-fade-leave{
        transform: translateY(0px);
        opacity: 1;
    }

</style>
