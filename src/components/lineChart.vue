<template>
  <div ref="lineDom" style="width: 900px; height: 400px"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from "vue";
//  按需引入 echarts
import * as echarts from "echarts";
const lineDom = ref() // 使用ref创建虚拟DOM引用，使用时用lineDom.value
const dataList = ref()
dataList.value = [
  {
    name: 'Email',
    type: 'line',
    stack: 'Total',
    data: [120, 132, 101, 134, 90, 230, 210]
  },
  {
    name: 'Union Ads',
    type: 'line',
    stack: 'Total',
    data: [220, 182, 191, 234, 290, 330, 310]
  },
  {
    name: 'Video Ads',
    type: 'line',
    stack: 'Total',
    data: [150, 232, 201, 154, 190, 330, 410]
  },
  {
    name: 'Direct',
    type: 'line',
    stack: 'Total',
    data: [320, 332, 301, 334, 390, 330, 320]
  },
  {
    name: 'Search Engine',
    type: 'line',
    stack: 'Total',
    data: [820, 932, 901, 934, 1290, 1330, 1320]
  }
]
onMounted(
  () => {
    initLine()
  }
)
function initLine() {
  // 基于准备好的dom，初始化echarts实例
  var myChart = echarts.init(lineDom.value);
  // 指定图表的配置项和数据
  var optionLine = {
    title: {
      text: 'Stacked Line'
    },
    tooltip: {
      trigger: 'axis'
    },
    legend: {
      data: ['Email', 'Union Ads', 'Video Ads', 'Direct', 'Search Engine']
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    toolbox: {
      feature: {
        saveAsImage: {}
      }
    },
    xAxis: {
      type: 'category',
      boundaryGap: false,
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
      type: 'value'
    },
    series: dataList.value
  };

  // 使用刚指定的配置项和数据显示图表。
  myChart.setOption(optionLine);
}
</script>


<style scoped>
div {
  display: inline-block;
  padding-left: 20px;
}
</style>

