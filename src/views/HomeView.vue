<!--
 * @Date: 2024-09-18 08:59:04
 * @LastEditors: aliushen 774779341@qq.com
 * @LastEditTime: 2024-09-19 23:17:17
 * @FilePath: \schedule\src\views\HomeView.vue
-->
<template>
  <div class="home">
    <p>
      计划下单时间：<el-date-picker v-model="nowDay" placeholder="选定日期" @change="planDateChange">
      </el-date-picker>
    </p>
    <input
      class="add-file"
      ref="inputFile"
      id="inputFile"
      type="file"
      @change="handle"
    />
    <el-button @click="handleDays" type="primary">下载结果</el-button>
    <el-button @click="reloading" type="primary">刷新页面</el-button>
    <el-table :data="resultArr" border>
      <el-table-column
        align="center"
        prop="name"
        label="人员姓名"
      >
      </el-table-column>
      <el-table-column
        align="center"
        prop="nowDays"
        label="工作量总和"
      >
      </el-table-column>
      <el-table-column
        align="center"
        prop="chaDays"
        label="距离下单到期日天数"
      >
      </el-table-column>
      <el-table-column align="center" prop="planDays" label="计划下单工作量">
      </el-table-column>
      <el-table-column
        align="center"
        prop="finishDays"
        label="上次非项目结算到期日"
      >
      </el-table-column>
      <el-table-column
        align="center"
        prop="trueDays"
        label="实际非项目结算到期日"
      >
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import * as XLSX from "xlsx/xlsx.mjs";
import { saveAs } from "file-saver";
import dayjs from "dayjs";
import { holidays, exchangeDays } from "../assets/holiday.js";
export default {
  name: "HomeView",
  data() {
    return {
      handleClick: false,
      nowDay: new Date("2024-11-29"),
      saveData: [],
      resultArr: [],
    };
  },
  mounted() {
    let days = this.addWorkDays(new Date("2024-03-02"), 5);

    console.log("答案是多少？" + dayjs(days).format("YYYY-MM-DD"));
  },
  methods: {
    planDateChange(e){
      
      this.resultArr=this.resultArr.map(item=>{
        let planDay = item["finishDays"];
        let orderDays = this.getWorkingDays(planDay, this.nowDay);
        let cha = orderDays - item["nowDays"];
        let tureDays = this.addWorkDays(
          new Date(planDay),
          Number(item["nowDays"])
        );
        return {
          name:item.name,
          nowDays:item.nowDays,
          chaDays:cha,
          planDays:orderDays,
          finishDays:item.finishDays,
          trueDays:this.timeFilter(tureDays)

        }
      })
      
    },
    reloading(){
      window.location.reload()
    },
    // 初始时间+工作日=最终时间
    addWorkDays(startDate, days) {
      let date = new Date(startDate);
      let addedDays = 0;
      // 判断startDate是否是工作日

      while (
        holidays.includes(dayjs(date).format("YYYY-MM-DD")) ||
        ((date.getDay() === 0 || date.getDay() === 6) &&
          !exchangeDays.includes(dayjs(date).format("YYYY-MM-DD")))
      ) {
        date.setDate(date.getDate() + 1);
        console.log("执行循环了？");
      }
      // 默认startDate当天也算工作日，所以减去,所以需要提前判断工作日。
      while (addedDays < days - 1) {
        date.setDate(date.getDate() + 1);
        addedDays++;
        // 检查是否为假期，如果是，就
        console.log(dayjs(date).format("YYYY-MM-DD"));
        if (holidays.includes(dayjs(date).format("YYYY-MM-DD"))) {
          addedDays--;
          // 如果是工作日，说明是对的，不用回减
        } else if (exchangeDays.includes(dayjs(date).format("YYYY-MM-DD"))) {
        } else if (date.getDay() === 0 || date.getDay() === 6) {
          // 如果是周末则减去一个工作日
          addedDays--;
        }
      }

      return date;
    },
    saveExcel(arr) {
      // 创建工作簿
      const wb = XLSX.utils.book_new();

      // 将数据转换为工作表
      const ws = XLSX.utils.aoa_to_sheet(arr);

      // 将工作表添加到工作簿
      XLSX.utils.book_append_sheet(wb, ws, "Sheet1");

      // 生成Excel文件
      const wbout = XLSX.write(wb, { bookType: "xlsx", type: "binary" });

      // 字符串转ArrayBuffer
      function s2ab(s) {
        const buf = new ArrayBuffer(s.length);
        const view = new Uint8Array(buf);
        for (let i = 0; i !== s.length; ++i) view[i] = s.charCodeAt(i) & 0xff;
        return buf;
      }

      // 保存文件
      saveAs(
        new Blob([s2ab(wbout)], { type: "application/octet-stream" }),
        "example.xlsx"
      );
    },
    handleDays() {
      console.log(this.nowDay);
      let arr=[]
      arr[0]=this.saveData[0]
      this.resultArr.map(item=>{
        arr.push([item.name,item.nowDays,item.chaDays,item.planDays,item.finishDays,item.trueDays])
      })
      this.saveExcel(arr);
    },
    timeFilter(time) {
      return dayjs(time).format("YYYY/MM/DD");
    },
    readFile(file) {
      return new Promise((resolve) => {
        let reader = new FileReader();
        reader.readAsBinaryString(file);
        reader.onload = (ev) => {
          resolve(ev.target.result);
        };
      });
    },

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
        } else if (
          !holidays.includes(dayjs(currentDate).format("YYYY-MM-DD")) &&
          dayOfWeek !== 0 &&
          dayOfWeek !== 6
        ) {
          totalDays++;
        }
        currentDate.setDate(currentDate.getDate() + 1);
      }
      return totalDays;
    },
    async handle(ev) {
      this.resultArr = [];
      this.handleClick = false;
      let arr = [
        [
          "人员姓名",
          "工作量总和",
          "距离下单到期日天数",
          "计划下单工作量",
          "上次非项目结算到期日",
          "实际非项目结算到期日"
        ],
      ];

      let file = ev.target.files[0];
      this.arr = [];
      if (!file) {
        console.log("失败");
        return;
      } else {
        let loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        let data = await this.readFile(file);
        let workbook = XLSX.read(data, { type: "binary" });
        // console.log(workbook);
        let worksheet = workbook.Sheets[workbook.SheetNames[0]]; //第一个sheet
        let result = XLSX.utils.sheet_to_json(worksheet);
        console.log("解析结果", result);

        result.map((item, index) => {
          // 获取
          if (index != 0 && item["工作量总和"]) {
            //不取第一行
            // 工作量总和
            let name=item['人员姓名']
            let allWorkDays = item["工作量总和"];

            // 一季度实际下单，不能动
            let planDay = item["上次非项目结算到期日"];
            // 计划下单（准确的）
            let orderDays = this.getWorkingDays(planDay, this.nowDay);
            let cha = orderDays - item["工作量总和"];
            let tureDays = this.addWorkDays(
              new Date(item["上次非项目结算到期日"]),
              Number(item["工作量总和"])
            );

            arr.push([
              name,
              item["工作量总和"],
              cha,
              orderDays,
              planDay,
              this.timeFilter(tureDays),
            ]);
            this.resultArr.push({
              name,
              nowDays: item["工作量总和"],
              chaDays: cha,
              planDays: orderDays,
              finishDays: planDay,
              trueDays: this.timeFilter(tureDays),
            });
          }
        });

        this.handleClick = true;
        loading.close();
        console.log(arr);

        this.saveData = arr;
        this.$refs.inputFile.value = "";
      }
    },
  },
};
</script>
