<!--
 * @Date: 2024-09-18 08:59:04
 * @LastEditors: aliushen 774779341@qq.com
 * @LastEditTime: 2024-09-19 19:21:49
 * @FilePath: \schedule\src\App.vue
-->
<template>
  <div id="app">
    <p style="padding-top:10px"> {{thisYear}}年假期安排</p>
    <div style="padding-bottom:30px">
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
    <router-view />
  </div>
</template>
<script>

export default {
  data(){
    return {
      thisYear:new Date().getFullYear(),
      thisYearData: [
        {
          name: "元旦",
          holiday: "12月30日-1月1日",
          days: 3,
          state: "已过",
        },
        {
          name: "春节",
          holiday: "2月10日-2月16日",
          days: 8,
          state: "已过",
        },
        {
          name: "清明节",
          holiday: "4月4日-4月6日",
          days: 3,
          state: "已过",
        },
        {
          name: "劳动节",
          holiday: "5月1日-5月5日",
          days: 5,
          state: "已过",
        },
        {
          name: "端午节",
          holiday: "6月9日-6月11日",
          days: 3,
          state: "已过",
        },
        {
          name: "中秋节",
          holiday: "9月15日-9月17日",
          days: 3,
          state: "已过",
        },
        {
          name: "国庆节",
          holiday: "10月1日-10月7日",
          days: 7,
          state: "未到",
        },
      ],
    }
  }
}
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}
</style>
