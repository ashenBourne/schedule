<template>
  <div>
    <p class="title">计算两个日期间工作日多少，当前或选定日期的进度为多少</p>
    <header>
      <p>初始、截止日期</p>
      <el-date-picker v-model="daysInterval" type="daterange" range-separator="至" start-placeholder="开始日期"
        end-placeholder="结束日期">
      </el-date-picker>
      <p>选定日期</p>
      <el-date-picker v-model="nowDay" placeholder="选定日期">
      </el-date-picker>
      <div>
        <el-button type="primary" @click="getDays">主要按钮</el-button>
      </div>
    </header>
    <div>
      <el-table :data="thisYearData" style="width: 100%" border>
        <el-table-column align="center" prop="name" label="节日名称">
        </el-table-column>
        <el-table-column align="center" prop="holiday" label="放假时间">
        </el-table-column>
        <el-table-column align="center" prop="days" label="天数">
        </el-table-column>
        <el-table-column align="center" prop="state" label="状态">
        </el-table-column>
      </el-table>
    </div>

    <p>
      当前选定时间为：{{ timeFilter(nowDay) }}
    </p>
    <p>
      当前时间段内共有<span class="emphasize">{{ workDays }}</span>个工作日，已完成了<span class="emphasize">{{ finishDays }}</span>个工作日
    </p>
    <p>
      进度为：<span class="emphasize">{{ schedule }}</span>
    </p>

  </div>
</template>

<script>
import dayjs from 'dayjs'
import { holidays, exchangeDays } from "../assets/holiday.js"
export default {
  data() {
    return {
      workDays: 0,
      finishDays: 0,
      schedule: '0%',
      daysInterval: "",
      nowDay: new Date(),

      thisYearData: [
        {
          "name": "元旦",
          "holiday": "12月30日-1月1日",
          "days": 3,
          "state": "已过"
        },
        {
          "name": "春节",
          "holiday": "2月10日-2月16日",
          "days": 8,
          "state": "已过"
        },
        {
          "name": "清明节",
          "holiday": "4月4日-4月6日",
          "days": 3,
          "state": "已过"
        },
        {
          "name": "劳动节",
          "holiday": "5月1日-5月5日",
          "days": 5,
          "state": "已过"
        },
        {
          "name": "端午节",
          "holiday": "6月9日-6月11日",
          "days": 3,
          "state": "已过"
        },
        {
          "name": "中秋节",
          "holiday": "9月15日-9月17日",
          "days": 3,
          "state": "正在进行"
        },
        {
          "name": "国庆节",
          "holiday": "10月1日-10月7日",
          "days": 7,
          "state": "未到"
        }
      ]
    }
  },
  mounted() {
    // 示例用法
    let start = new Date('2024-09-01');
    let end = new Date('2024-09-30');
    let holidays = [new Date('2024-09-17').getTime()];
    let workingDays = this.getWorkingDays(start, end, holidays);
    console.log(workingDays);
  },
  methods: {
    getDays() {
      if (this.daysInterval && this.daysInterval.length == 2) {
        // 总工作日
        this.workDays = this.getWorkingDays(new Date(this.daysInterval[0]), new Date(this.daysInterval[1]))
        // 已完成工作日
        this.finishDays = this.getWorkingDays(new Date(this.daysInterval[0]), this.nowDay)
        // 进度
        this.schedule = Number(this.finishDays / this.workDays).toFixed(2) * 100 + '%'
      }
    },
    timeFilter(time) {
      return dayjs(time).format('YYYY-MM-DD')
    },
    // 参数均为时间戳
    getWorkingDays(startDate, endDate) {
      let totalDays = 0;
      let currentDate = new Date(startDate);
      endDate = new Date(endDate);
      while (currentDate <= endDate) {
        let dayOfWeek = currentDate.getDay();
        // 如果是调休，直接算工作日
        if (exchangeDays.includes(dayjs(currentDate).format("YYYY-MM-DD"))) {
          totalDays++;

          // 如果非节假日，且非周六日，算工作日
        } else if (!holidays.includes(dayjs(currentDate).format("YYYY-MM-DD")) && dayOfWeek !== 0 && dayOfWeek !== 6) {
          totalDays++;

        }
        currentDate.setDate(currentDate.getDate() + 1);
      }
      return totalDays;
    },

  }
}
</script>

<style  scoped>
.emphasize {
  color: red;
  font-size: 20px;
  font-weight: 900;
}
</style>