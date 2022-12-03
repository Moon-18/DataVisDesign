<template>
  <h2 align="center" class="title">用户个人支出财务统计及全国信息对比</h2>
  <div style="width:1200px;">
    <span ref="inputBox">
      <!-- <el-input v-model="inputSeason" class="inputTxt" placeholder="输入季度名称" :suffix-icon="Calendar" /> -->
      <el-cascader v-model="inputSeason" :options="seasonOption" placeholder="选择季度名称" />
      <!-- <el-input v-model="inputName" class="inputTxt" placeholder="输入支出种类" :suffix-icon="Goods" /> -->
      <el-cascader v-model="inputName" :options="nameOption" placeholder="选择支出种类" />
      <el-input v-model="inputValue" class="inputTxt" placeholder="输入支出数值" :suffix-icon="Search" />
    </span>
    <el-button type="primary" @click="addData()">增加个人数据</el-button>
    <el-button type="danger" @click="clearData()">清空个人数据</el-button>
    <el-button type="success" @click="showSummary">生成个人报告</el-button>
    <el-button type="info">
      <a href="http://gdzd.stats.gov.cn/gzdcd/gz_tjsj/" target="_blank" style="text-decoration: none">数据来源</a>
    </el-button>
  </div>
  <el-carousel autoplay=false :interval="50000" indicator-position="none" trigger="click" arrow="always" height="660px"
    @change="((pre: any, next: any) => { handleChange(pre, next) })">
    <el-carousel-item>
      <div ref="pieDom" class="chart"></div>
    </el-carousel-item>
    <el-carousel-item>
      <div ref="lineDom" class="chart"></div>
    </el-carousel-item>
    <el-carousel-item>
      <iframe src="graph.html" frameborder="0" width="width: 1200px" scrolling="no"
        style="width: 100%; height: 100%;background-color: white;margin:auto"></iframe>
    </el-carousel-item>
    <el-carousel-item>
      <iframe src="pictorialBar-bar-transition.html" frameborder="0" width="width: 1200px" scrolling="no"
        style="width: 100%; height: 100%;background-color: white;margin:auto"></iframe>
    </el-carousel-item>
  </el-carousel>


</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import * as echarts from "echarts";//  按需引入 echarts
import { ElMessage, ElMessageBox } from 'element-plus'
import type { Action } from 'element-plus'
import { Calendar, Search, Goods } from '@element-plus/icons-vue'
import lineChartVue from "./components/lineChart.vue";
const inputBox = ref()
function handleChange(index: any, index2: any) {//TODO 显示后两幅的时候禁止输入...不做了...
  // window.alert(index)
  // if (index == 2) {
  //   inputBox.value.display
  // }
}
const inputName = ref('')
const inputSeason = ref()
const inputValue = ref()
const seasonOption = [{ value: "第一季度", label: "第一季度" }, { value: "第二季度", label: "第二季度" }, { value: "第三季度", label: "第三季度" }, { value: "第四季度", label: "第四季度" }]
const nameOption = [{ value: "住房支出", label: "住房支出" }, { value: "饮食支出", label: "饮食支出" }, { value: "交通支出", label: "交通支出" },
{ value: "教育支出", label: "教育支出" }, { value: "医疗支出", label: "医疗支出" }, { value: "娱乐支出", label: "娱乐支出" }, { value: "其他支出", label: "其他支出" }]

function addData() {
  //TODO2:增加数据,数据的来源是上面这仨变量,需要添加到yearData, seasonData的对应项中
  //inputName在已有标签中,添加对应数据; 否则添加到其他
  var finded = false;
  for (let i = 0; i < yearData.value.length; i++) {
    let element = yearData.value[i];
    if (inputName.value[0] === element.name) {
      finded = true;
      // element.value += inputValue.value;//这样写有问题 直接以字符串相加了
      // element.value += Number(inputValue);//不行
      element.value += Number(inputValue.value);
      break;
    }
    console.log(element);
  }
  if (finded === true) {
    for (let i = 0; i < seasonData.value.length; i++) {
      // const element = seasonData.value[i]; //不能使用常量const
      if (seasonData.value[i].name === inputName.value[0]) {
        if (inputSeason.value[0] === '第一季度') seasonData.value[i].data[0] += Number(inputValue.value);
        else if (inputSeason.value[0] === '第二季度') seasonData.value[i].data[1] += Number(inputValue.value);
        else if (inputSeason.value[0] === '第三季度') seasonData.value[i].data[2] += Number(inputValue.value);
        else if (inputSeason.value[0] === '第四季度') seasonData.value[i].data[3] += Number(inputValue.value);
        break;
      }
    }
  }
  else {//加到"其他"一类中
    yearData.value[6].value += Number(inputValue.value);
    if (inputSeason.value[0] === '第一季度') seasonData.value[6].data[0] += Number(inputValue.value);
    else if (inputSeason.value[0] === '第二季度') seasonData.value[6].data[1] += Number(inputValue.value);
    else if (inputSeason.value[0] === '第三季度') seasonData.value[6].data[2] += Number(inputValue.value);
    else if (inputSeason.value[0] === '第四季度') seasonData.value[6].data[3] += Number(inputValue.value);
  }
  initPie();
  initLine();
  inputName.value = '';
  inputSeason.value = '';
  inputValue.value = '';
}
function clearData() {  //TODO 清空数据按钮 清空不了
  yearData.value.forEach((item: any) => { item.value = 0 });
  seasonData.value.forEach((item: any) => {
    // item.data.forEach((it2: any) => { it2 = 0 }) 
    // console.log(item.data)
    item.data = [0, 0, 0, 0]
  });
  initLine();
  initPie();
}
const summaryStr = ref('')
function createSummary() {
  //TODO3 根据数据补充一下内容,大概就是遍历一下俩数组,分析点有用的特征,然后把下面的写写
  let maxYearExpensePos = -1, maxYearExpense = -1, rate = 0.0, sumExpense = 0;
  for (let i = 0; i < yearData.value.length; i++) {
    let item = yearData.value[i];
    sumExpense += item.value;
    if (item.value > maxYearExpense) {
      maxYearExpense = item.value;
      maxYearExpensePos = i;
    }
  }
  rate = 1.0 * maxYearExpense / sumExpense * 100.0;
  summaryStr.value = "在您的全年支出中, ";
  summaryStr.value += "最高支出项是" + yearData.value[maxYearExpensePos].name + ", ";
  summaryStr.value += "共花费" + maxYearExpense.toString() + "元, 占比总支出" + rate.toFixed(2) + "%。\n";

  let season = ['第一季度', '第二季度', '第三季度', '第四季度'];
  let maxSeasonExpense = -1, maxSeasonExpensePos = -1, minSeasonExpense = 1000000000, minSeasonExpensePos = -1;
  let seasonExpense = [0, 0, 0, 0];
  for (let i = 0; i < seasonData.value.length; i++) {
    let item = seasonData.value[i];
    seasonExpense[0] += item.data[0];
    seasonExpense[1] += item.data[1];
    seasonExpense[2] += item.data[2];
    seasonExpense[3] += item.data[3];
  }
  for (let i = 0; i < 4; i++) {
    if (seasonExpense[i] < minSeasonExpense) {
      minSeasonExpense = seasonExpense[i];
      minSeasonExpensePos = i;
    }
    if (seasonExpense[i] > maxSeasonExpense) {
      maxSeasonExpense = seasonExpense[i];
      maxSeasonExpensePos = i;
    }
  }
  summaryStr.value += "在您的季度支出中, 支出最高的季度是" + season[maxSeasonExpensePos] + ", 共支出" + maxSeasonExpense.toString() + "元; ";
  summaryStr.value += "支出最低的季度是" + season[minSeasonExpensePos] + ", 共支出" + minSeasonExpense.toString() + "元。\n";
  summaryStr.value += "2021年全国的人均支出为24100元, 您的总支出远远高于此数字。鉴于您所在地为广州, 生活成本普遍较高, 建议您合理消费, 注意理财。祝您生活如意, 步步高升。\n";

}
const showSummary = () => {
  createSummary()
  console.log(summaryStr.value)
  ElMessageBox.alert(summaryStr.value, '个人报告', {
    // if you want to disable its autofocus
    // autofocus: false,
    confirmButtonText: 'OK',
    callback: (action: Action) => {
      ElMessage({
        type: 'success',
        message: `成功`,
      })
    },
  })
}
//TODO1:合理数据,符合对应项 全年=季度之和,且符合正常的支出(自己搜搜或造一造)
const yearData = ref([] as any)//饼图数据,数据含义是,这一项全年的支出
yearData.value = [
  { value: 11059, name: '住房支出' },
  { value: 14974, name: '饮食支出' },
  { value: 6223, name: '交通支出' },
  { value: 6145, name: '教育支出' },
  { value: 2179, name: '医疗支出' },
  { value: 2853, name: '娱乐支出' },
  { value: 1502, name: '其他支出' }
]

const seasonData = ref()//折线图部分数据,数据每一项中data含义是,这一项这一季度的支出(0,1,2,3季度)
seasonData.value = [//采样自2021年广州城镇居民季度支出
  {
    name: '住房支出',
    type: 'line',
    //stack: 'Total',
    data: [3014, 2831, 2986, 2228]
  },
  {
    name: '饮食支出',
    type: 'line',
    //stack: 'Total',
    data: [4038, 3665, 3693, 3578]
  },
  {
    name: '交通支出',
    type: 'line',
    //stack: 'Total',
    data: [1560, 1472, 1713, 1478]
  },
  {
    name: '教育支出',
    type: 'line',
    //stack: 'Total',
    data: [1552, 1505, 1750, 1338]
  },
  {
    name: '医疗支出',
    type: 'line',
    //stack: 'Total',
    data: [504, 687, 612, 376]
  },
  {
    name: '娱乐支出',
    type: 'line',
    //stack: 'Total',
    data: [763, 687, 727, 676]
  },
  {
    name: '其他支出',
    type: 'line',
    //stack: 'Total',
    data: [464, 353, 290, 395]
  }
]


//饼图配置部分
const pieDom = ref() // 使用ref创建虚拟DOM引用，使用时用pieDom.value
var myPieChart: any;
const optionPie = ref({
  tooltip: {
    trigger: 'item'
  },
  legend: {
    // top: '10%',
    bottom: "5%",
    left: 'center'
  },
  series: [
    {
      name: 'Access From',
      type: 'pie',
      radius: ['40%', '70%'],
      avoidLabelOverlap: false,
      itemStyle: {
        borderRadius: 10,
        borderColor: '#fff',
        borderWidth: 2
      },
      label: {
        show: false,
        position: 'center'
      },
      emphasis: {
        label: {
          show: true,
          fontSize: '40',
          fontWeight: 'bold'
        }
      },
      labelLine: {
        show: false
      },
      data: yearData.value
    }
  ]
})
function initPie() {
  // 基于准备好的dom，初始化echarts实例
  // 指定图表的配置项和数据
  // 使用刚指定的配置项和数据显示图表。
  myPieChart.setOption(optionPie.value, true);
}

//折线图配置部分
const lineDom = ref() // 使用ref创建虚拟DOM引用，使用时用lineDom.value
var myLineChart: any;// 基于准备好的dom，初始化echarts实例
const optionLine = ref({
  title: {
    text: ''
  },
  tooltip: {
    trigger: 'axis'
  },
  legend: {
    data: ['住房支出', '饮食支出', '交通支出', '教育支出', '医疗支出', '娱乐支出', '其他支出']
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
    data: ['第一季度', '第二季度', '第三季度', '第四季度']
  },
  yAxis: {
    type: 'value'
  },
  series: seasonData.value
});
function initLine() {
  // 指定图表的配置项和数据
  myLineChart.setOption(optionLine.value, true);// 使用刚指定的配置项和数据显示图表。
}
onMounted(
  () => {
    myPieChart = echarts.init(pieDom.value);//注意声明周期钩子
    myLineChart = echarts.init(lineDom.value);
    initPie()
    initLine()
  }
)
</script>


<style scoped>
.inputTxt {
  width: 200px;
  padding: 10px;
}


div {
  margin: auto
}

.chart {
  width: 660px;
  height: 660px;
  background-color: white;
}

.title {
  margin: 0;
}
</style>

