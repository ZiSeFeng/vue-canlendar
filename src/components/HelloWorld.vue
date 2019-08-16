<template>
  <div class="canlandar">
    <h1>{{year}}-{{month}}</h1>
    <div class="tablebox1">
      <table class="bgtable">
        <thead>
          <tr>
            <th v-for="(item, index) in daynamearr" :key="index">{{item}}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(week, index) in theweek()" :key="index">
            <td v-for="(item, index2) in week" :key="index2" :class="{'gray': item.isMonth}">
              {{item.day}}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="tablebox2">
      <div class="row" v-for="(row, index) in domdata" :key="index">
        <table class="scheduletable">
          <tr v-for="(line, index2) in row" :key="index2">
            <td v-for="(td, index3) in line" :key="index3" :colspan="td.colspan">
              <div v-if="td.title" :class="['s', td.type]">{{td.title}}</div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import { Domain } from 'domain';
export default {
  data() {
    return {
      // 中文汉字星期名字
      daynamearr: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'],
      //本月的日期构成，先用二维数组
      dayarr: [],
      year: 2019,
      month: 8,
      domdata: [],
      schedules:[
        { 'title': '开会3天', 'start': '20190816', 'end': '20190816', 'type': 'life'},
        { 'title': '打球2天', 'start': '20190809', 'end': '20190810', 'type': 'life'},
        { 'title': '吃晚饭', 'start': '20190809', 'end': '20190809', 'type': 'life'},
        { 'title': '吃饭3天', 'start': '20190810', 'end': '20190812', 'type': 'life'},
        { 'title': '玩游戏4天', 'start': '20190817', 'end': '20190820', 'type': 'life'},
      ],
      daythingsnumber: [
        [0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0],
        [0,0,0,0,0,0,0]
      ]
    }
  },
  mounted() {
    let dayarr = [];
    // 需要知道这个月的第一天是星期几
    let themonthfirstday = (new Date(this.year, this.month-1, 1)).getDay();
    // 需要知道本月有多少天？ 此时经典的算法是判断这个月是不是1，3，5，7，8，10，12
    // 那么就是31天， 4、6、9、11有30天， 2月单独要看是不是闰年
    // 这个月的最后一天，就是下个月的第一天减1毫秒的那天
    let y = this.month === 12 ? this.year + 1 : this.year;
    let m = this.month === 12 ? 1 : this.month;
    // 日期对象减去常量表示得到时间戳，减去1表示减去1毫秒
    let themonthdaysamont = new Date((new Date(y, m, 1) - 1)).getDate();
    // 需要知道上一个月最后一天是几号？
    // 上一个月的最后一天是几号，就是这个月的第一天减去1毫秒
    var prevmonthlastdate = new Date(new Date(this.year, this.month -1, 1) -1).getDate();
    // 这个月的第一天星期几，就要放入几个上月的尾巴
    while (themonthfirstday-- > 0) {
      // dayarr.unshift(prevmonthlastdate--);
      dayarr.unshift({'day': prevmonthlastdate --, 'year': y, 'month': m, 'isMonth': true, 'occupation': []})
    };
    // 放置本月的日期
    let count = 1;
    while (themonthdaysamont--) {
      // dayarr.push(++count)
      dayarr.push({'day': count++, 'year': y, 'month': m, 'isMonth': false, 'isMonth': false, 'occupation': []});
    }
    // 补足35
    let count2 = 1;
    if (dayarr.length < 35) {
      while(dayarr.length !== 35) {
        // dayarr.push(++count2)
        dayarr.push({'day': count2++, 'year': y, 'month': m, 'isMonth': true, 'isMonth': true, 'occupation': []})
      }
    } else {
      while(dayarr.length !== 42) {
        // dayarr.push(++count2)
        dayarr.push({'day': count2++, 'year': y, 'month': m, 'isMonth': true, 'isMonth': true, 'occupation': []})
      }
    }
    this.dayarr = dayarr;

    this.calcDomData(dayarr, this.schedules, this.daythingsnumber);
  },
  methods: {
    theweek() {
      if (this.dayarr.length === 0) return [];
      var _arr = [[],[],[],[],[],[]];
      for (let i = 0; i < this.dayarr.length / 7; i++) {
        var _week = [];
        for (let j = 0; j < 7; j++) {
          _week.push(this.dayarr[i * 7 + j])
        }
        _arr.push(_week);
      }
      return _arr;
    },
    calcDomData: function (dayarr, schedules) {
          /* 将后台给的schedules 使用函数转换成三维数组，再使用vue 在dom中循环三维数组
          1.返回的数组中，每一项代表哪天？
          2.每一个日程，需要选择哪个line？
          3.添加colspan的小格，需要删除多余的小格
          4.跨周的做法？*/
          var domdata = dayarr.length == 42 ? [
              [[], [], []],
              [[], [], []],
              [[], [], []],
              [[], [], []],
              [[], [], []],
              [[], [], []]
          ] : [
              [[], [], []],
              [[], [], []],
              [[], [], []],
              [[], [], []],
              [[], [], []]
          ];
          //遍历每一行，将dayarr 的35项的数组变成以周为单位的小数组
          for (let rowindex = 0; rowindex < dayarr.length / 7; rowindex++) {
              let weekarr = dayarr.slice(rowindex * 7, (rowindex + 1) * 7);
              weekarr.forEach(function (theday, index) {
                  var thedayschedules = []; //这一天要做的事情
                  //探访所有事件，将所有当天发生的事情推入以上的数组
                  //遍历所有事件
                  schedules.forEach(function (theschedule, index2) {
                      //事件开始年月日
                      var startyear = theschedule.start.slice(0, 4);
                      var startmonth = theschedule.start.slice(4, 6);
                      var startday = theschedule.start.slice(6, 8);
                      //事件结束年月日
                      var endyear = theschedule.end.slice(0, 4);
                      var endmonth = theschedule.end.slice(4, 6);
                      var endday = theschedule.end.slice(6, 8);
                      //事件持续时间
                      theschedule.colspan = (new Date(endyear, endmonth - 1, endday) - new Date(startyear, startmonth - 1, startday)) / (1000 * 24 * 60 * 60) + 1;
                      //这个事是不是当天发生
                      if (startyear == theday.year && startmonth == theday.month && startday == theday.day) {
                          thedayschedules.push(theschedule);
                      }
                  });
                  //遍历3个line 事件
                  for (let line = 0; line < 3; line++) {
                      if (theday.occupation.indexOf(line) == -1) {
                          var first = thedayschedules.shift(); //出栈
                          if (first === undefined) {
                              domdata[rowindex][line].push({"title": "", "colspan": 1});
                          } else {
                              domdata[rowindex][line].push({
                                  "title": first.title,
                                  "type": first.type,
                                  "colspan": first.colspan,
                                  "norightradius": first.colspan > 7
                              });
                              //将这一天和后面的colspan的天数的occupation中加入自己的行号
                              if (first.colspan && first.colspan != 1) {
                                  //end 事件结束的星期号  index这天的星期数  colspan这天发生的第一件事的持续天数
                                  let end = index + first.colspan - 1 > 6 ? 6 : index + first.colspan - 1;
                                  //除去本周后，还剩余的天数，比如周三开始持续8天的事件，还剩4天需要在之后的周别安排
                                  let rest = first.colspan - (7 - index);
                                  console.log(end);
                                  for (let _i = index + 1; _i <= end; _i++) {
                                      weekarr[_i].occupation.push(line);
                                  }
                                  var _rest;
                                  var count = 0; //数量，与当前正在遍历的周差了几周
                                  while (rest > 0) {
                                      _rest = rest > 7 ? 7 : rest;
                                      rest -= _rest;
                                      count++;
                                      //下一周
                                      var nextweek = dayarr.slice((rowindex + count) * 7, (rowindex + count) * 7 + 7);
                                      if (nextweek == undefined) {
                                          continue;
                                      }
                                      domdata[rowindex + count][line].push({
                                          "title": first.title,
                                          "type": first.type,
                                          "colspan": _rest,
                                          "norightradius": rest != 0,
                                          "noleftradius": true
                                      });
                                      for (let j = 0; j < 7; j++) {
                                          nextweek[j].occupation.push(line);
                                      }
                                  }
                              }
                          }
                      }
                  }
              });
          }
          this.domdata = domdata;
      },
  },
}
</script>

<style lang="scss" scoped>
* {
    margin: 0;
    padding: 0;
}
h4 {
    position: absolute;
    top: -14px;
    left: -60px;
    font-size: 24px;
}
.canlender {
    height: 530px;
    width: 1000px;
    /*border: 1px solid #333;*/
    margin: 30px auto;
    position: relative;
}
table.bgtable {
    width: 100%;
    border-collapse: collapse;
}
table.bgtable thead tr th {
    height: 30px;
}
table.bgtable tbody tr td {
    height: 100px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    text-align: right;
    vertical-align: top;
    padding: 5px;
}
table.bgtable tbody tr .gray {
    background: #eee;
}
.tablebox2 {
    position: absolute;
    top: 30px;
    width: 1000px;
    height: 600px;
    padding-top: 30px;
}
.tablebox2 .row {
    height: 100px;
    box-sizing: border-box;
}
.tablebox2 .row .scheduletable {
    width: 100%;
    height: 70px;
}
.tablebox2 .row .scheduletable tr {
    width: 100%;
    height: 20px;
}
.tablebox2 .row .scheduletable tr td {
    width: 14.2857%;
    box-sizing: border-box;
}
.tablebox2 .row .scheduletable tr .s {
    border-radius: 15px;
    padding-left: 10px;
    font-size: 14px;
    line-height: 20px;
}
.tablebox2 .row .scheduletable tr .norightradius {
    border-bottom-right-radius: 0px;
    border-top-right-radius: 0px;
}
.tablebox2 .row .scheduletable tr .noleftradius {
    border-bottom-left-radius: 0px;
    border-top-left-radius: 0px;
}
.tablebox2 .row .scheduletable tr .life {
    background: yellowgreen;
}
.tablebox2 .row .scheduletable tr .work {
    background: dodgerblue;
}
.tablebox2 .row .scheduletable tr .important {
    background: orangered;
}
</style>