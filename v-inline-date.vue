<template>
  <div>
    <div class="sl-select-date">

      <div class="sl-ul">
        <div class="sl-li">日</div>
        <div class="sl-li">一</div>
        <div class="sl-li">二</div>
        <div class="sl-li">三</div>
        <div class="sl-li">四</div>
        <div class="sl-li">五</div>
        <div class="sl-li">六</div>
      </div>
      <div class="sl-loading" v-if="!pullDate">
        加载中...
      </div>
      <template v-else>
        <div v-for="(item, index) in pullDate" :key="index">
          <div class="sl-date-title">{{item.title}}</div>
          <div class="sl-ul">
            <div class="sl-li" v-for="(day,i) in item.date" :class="day.classname" :key="i"
                @click="dayClickEvent(day)">
              <div :class="
              (currentdate[0].date!==''&&currentdate[0].date === day.date) || 
              (currentdate[1]&&currentdate[1].date!==''&&currentdate[1].date === day.date)?
              'sl-curday':''">
                <span class="sl-day-txt" :class="currentdate[0].str || currentdate[1]&&currentdate[1].str?'sl-hasstr':''"> 
                  {{day.day}} <span class="sl-circle"  v-if="day.classname && day.classname.indexOf('sl-today')!==-1"></span>
                </span>
                
                <!--cur-tag-->
                <span class="sl-cur-str" v-if="currentdate[0].date!==''&&currentdate[0].date === day.date && currentdate[0].str"> {{currentdate[0].str}}</span>
                <span class="sl-cur-str" v-else-if="currentdate[1]&&currentdate[1].date!==''&&currentdate[1].date === day.date && currentdate[1].str"> {{currentdate[1].str}}</span>
                
                <!--tag-->
                <span class="sl-i" v-for="tag in day.tags" :class="tag[0]" :key="tag[0]">
                  {{tag[1]}}<span class="sl-span" v-if="tag[0]==='price'">元</span>
                </span>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>

  </div>
</template>

<script>
  import moment from 'moment';
  /**
   * dateJson:[{ date: "2019-8-13",price: "100",discount: "折",}]; // discount自定义,对应的class名字也叫discount
   * current: 三种传值方式,数组的话最大可以传两个数据
       "2019-8-13",
       ["2019-8-13","2019-8-16"],
       [{date:moment().add(1,'day').format('YYYY-MM-DD'), str:'入店'}]
   */
  export default {
    name:'v-inline-date',
    props:{
       startDate:{type:String,default:moment().format('YYYY-MM-DD')},
       endDate:{type:String,default:moment().add(1,'year').format('YYYY-MM-DD')},
       dateJson:{type:Array,default:  () => []},
       current:{type:Array | String ,default: ''}, 
       dateType:{type:Number ,default: 0}
    },
    data() {
      return {
        pullDate: null,
        currentdate: [],
        currentTemp: [],
        count:0,
        selectArr:[],
      }
    },
    created() {
     this.init();
    },
    methods: {
      init(){
        this.$nextTick(() => {
          this.pullDate = this.getAll(this.startDate, this.endDate);

          if(this.dateType === 0){
            if(typeof this.current === 'string'){
              this.currentdate = [{date:this.current,str:''}]
            }else{
              this.currentdate = [{date:this.current[0],str:''}]
            }
          }
          if(this.dateType === 1){
            if(typeof this.current === 'string'){
              this.currentdate = [{date:'',str:''},{date:'',str:''}];
              console.error('current类型为数组,长度为2!');
            }else{
              this.currentdate = []
              this.current.forEach(item => {
                if(typeof item === 'string'){
                  this.currentdate.push({date:item,str:''})
                }else{
                this.currentdate.push(item)
                }
              })
            }
          }
          this.currentTemp = this.currentdate;
        })
      },
      dayClickEvent(data) {
        if(data.classname && data.classname.indexOf('sl-day')===-1){return}
        if(this.dateType === 1) { 
          if(this.count >=1){
            if(moment(data.date).diff(moment(this.currentdate[0].date)) <= 0){
            this.currentdate = [];
            this.currentdate[0] = this.currentTemp[0]
              this.currentdate[0].date = data.date;
              this.selectArr = [data];
              return;
            }
            this.selectArr.push(data);
            this.count = 0;
            this.currentdate[1] = this.currentTemp[1]
            this.currentdate[1].date = data.date;
            this.$emit('selectDate', this.selectArr);
            this.selectArr = [];
            return;
          }
          this.selectArr = [data];
          this.count++;
          this.currentdate = [];
          this.currentdate[0] = this.currentTemp[0]
          this.currentdate[0].date = data.date;
        }else{
          this.$emit('selectDate', data);
        }
      },
      pushTag(yearMonthDay) {
        let tags = [];
        for (let i = 0; i < this.dateJson.length; i++) {
          if (moment(yearMonthDay).format('x') === moment(this.dateJson[i].date).format('x')) {
            for (let key in this.dateJson[i]) {
              if(key!=='date'){
                tags.push([key, this.dateJson[i][key]]);
              }
            }
            break;
          }
        }
        return tags;
      },
      setClass(start, end, i) {
        let className = '',curstr = '';
        if (i >= moment(start).format('x') && i <= moment(end)) {
          className = 'sl-day';
          if (moment(i).format('YYYY-MM-DD') === moment().format('YYYY-MM-DD')) { 
            className += ' sl-today';
          }          
        } else {
          className = 'sl-pass';
          if (moment(i).format('YYYY-MM-DD') === moment().format('YYYY-MM-DD')) {
            className += ' sl-today';
          }
        }
        return {className,curstr};
      },
      getAll(start, end) {
        const sd = Number(moment(start).startOf('month').format('x')); 
        const ed = Number(moment(end).endOf('month').format('x')); 

        let dataObject = {};

        dataObject[sd] = {title: moment(start).format('YYYY年MM月'), date: []} 
        for (let w = 0; w < moment(sd).weekday(); w++) { 
          dataObject[sd].date.push({date:'',year: '', month: '', day: '', week: w,classname:'',curstr:'',tags:null});// 如果当前月份没有存储当前天数用的数组,就创建一个空数组，如果有，就向里面添加一个空对象; (空对象是用来占位置的，用来填充月份前面的空白)
        }

        for (let i = sd; i <= ed;) {
          const firstDay = Number(moment(i).startOf('month').format('x')); 

          if (moment(i).format('x') === moment(moment(i).startOf('month').format('YYYY-MM-DD')).format('x') && i !== sd) { // 如果是当月的第一天,添加下个月的数据
            const op = {
              title: moment(i).add(1, 'days').format('YYYY年MM月'), 
              date: []
            }
            for (let w = 0; w < moment(i).weekday(); w++) {
              op.date.push({date:'',year: '', month: '', day: '', week: w,classname:'',curstr:'',tags:null});
            }
            dataObject[i] = op;
          }

          let className = this.setClass(start, end, i).className;
          let curstr = this.setClass(start, end, i).curstr;

          const tag = this.pushTag(moment(i).format('YYYY-MM-DD')); 

          const option = {
            date: moment(i).format('YYYY-MM-DD'),
            year: moment(i).format('YYYY'),
            month: moment(i).format('MM'),
            day: moment(i).format('DD'),
            week: moment(i).weekday(),
            classname: className,
            curstr: curstr,
            tags: tag
          }
          dataObject[firstDay].date.push(option);
          i = Number(moment(i).add(1, 'days').format('x')); 
        }
        return dataObject;
      }
    }
  }
</script>

<style  lang="less" scoped>
 @curColor: #ff6600;
  @discountColor:#333;
  @priceColor:#333;
  div{box-sizing: border-box;}
 .sl-select-date
    {
    color: #333;
    .sl-date-title
      {width: 100%;
      background-color: transparent;
      color: #333;
      line-height: 70px;
      font-weight: 400;
      text-align: center;
      letter-spacing: 3px;
      font-size: 18px;
      z-index: 9}
      .sl-loading{padding: 100px;text-align: center;}
    }
    .sl-ul
      {overflow: hidden;
      border-bottom: 1px solid #e8e8e8;display: flex;flex-wrap: wrap;
      background: #fafafa}
      .sl-li
        {position: relative;
        width: 14%;height: 50px;
        margin-bottom: -1px;
        border-bottom: 1px solid #e8e8e8;
        text-align: center;
        line-height: 50px;
        font-size: 16px;
        
        &.sl-pass, &.sl-future
          {color: #ccc;
          opacity: 0.5}
        &.sl-day
          {color: #333}
          .sl-circle{display: none;}
        &.sl-today{color:#152281;position: relative;
          .sl-circle{position: absolute;display: block;width:5px;height:5px;border-radius: 5px;background: #152281;right: 3px;top:3px;}
        }
        .sl-curday{
          width:100%;height:100%;
          background: @curColor;
          .sl-hasstr{
            padding-top: 10px;
            display: block;
            line-height: 20px;
          }
          .sl-cur-str{font-size:12px;line-height: 19px;display: block;z-index: 9;position: relative;background: @curColor;}
        }
        .sl-i
          {
            display: block;
            position: absolute;
            color: #333;
            font-style: normal;
            font-size: 12px;line-height: 20px;
            width: 20px;
            height: 20px;z-index: 20;
            &.date
              {display: none;
              font-style: none}
            &.rest, &.discount
              {
              right: 0px;
              top: 5px;
              color:@discountColor}
            &.price
              {bottom: 0;
              left: 0;
              width: 100%;text-align: center;
              color: @priceColor;
              font-size: 12px;height: 20px;z-index: 1;
              line-height: 26px}
          }
        }

</style>




