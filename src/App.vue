<template>
  <div></div>
  <h2 align="center">用户个人支出财务统计及全国信息对比</h2>
  <div style="width: 50vw;">
    <el-input v-model="inputSeason" class="inputTxt" placeholder="输入季度名称" :suffix-icon="Calendar" />
    <el-input v-model="inputName" class="inputTxt" placeholder="输入支出名称" :suffix-icon="Goods" />
    <el-input v-model="inputValue" class="inputTxt" placeholder="输入支出数值" :suffix-icon="Search" />
    <el-button type="primary" @click="addData()">增加数据</el-button>
    <el-button type="success">生成报告</el-button>
  </div>
  <el-carousel :interval="10000" type="card" height="600px">
    <el-carousel-item>
      <div ref="pieDom" style="width: 900px; height: 600px;background-color: white;"></div>
    </el-carousel-item>
    <el-carousel-item>
      <div ref="lineDom" style="width: 900px; height: 600px;background-color: white;"></div>
    </el-carousel-item>
  </el-carousel>

  <div style="width: 50vw;display:inline-block">

    <!-- <div v-show="!choose" ref="lineDom" style="width: 900px; height: 600px;display:inline-block"></div> -->
  </div>



</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import * as echarts from "echarts";//  按需引入 echarts
import { Calendar, Search, Goods } from '@element-plus/icons-vue'

const inputName = ref('')
const inputSeason = ref()
const inputValue = ref()
function addData() {

  //TODO2:增加数据,数据的来源是上面这仨变量,需要添加到yearData,seasonData的对应项中
  //inputName在已有标签中,添加对应数据;否则添加到其他
  //大概长下面两条的样子
  yearData.value.push({ value: 50, name: 'rose 9' })
  seasonData.value[0].data[0] += 50
  initPie()
  initLine()
  console.log(yearData.value);
  inputName.value = ""
  inputSeason.value = 0
  inputValue.value = 0

}
const summaryStr = ref('')
function createSummary() {
  //TODO3 根据数据补充一下内容,大概就是遍历一下俩数组,分析点有用的特征,然后把下面的写写
  summaryStr.value = "在您的全年支出中,最大的是:XX支出,花费XX元,占比总支出XX%\n"
  summaryStr.value += "在您的季度支出中,XX支出,花费XX元,(写个大概的分析)\n"
  summaryStr.value += "20XX年,全国的人均支出为XXX(可以写死),与您的总支出对比,谁较高,消费支出习惯合理/不合理,建议以后理财或控制某方面支出啥的\n"
}
//TODO1:合理数据,符合对应项 全年=季度之和,且符合正常的支出(自己搜搜或造一造)
const yearData = ref([] as any)//饼图数据,数据含义是,这一项全年的支出
yearData.value = [
  { value: 40, name: '住房支出' },
  { value: 38, name: '饮食支出' },
  { value: 32, name: '交通支出' },
  { value: 30, name: '教育支出' },
  { value: 28, name: '医疗支出' },
  { value: 26, name: '娱乐支出' },
  { value: 18, name: '其他支出' }
]

const seasonData = ref()//折线图部分数据,数据每一项中data含义是,这一项这一季度的支出(0,1,2,3季度)
seasonData.value = [
  {
    name: '住房支出',
    type: 'line',
    //stack: 'Total',
    data: [120, 132, 101, 134]
  },
  {
    name: '饮食支出',
    type: 'line',
    //stack: 'Total',
    data: [220, 182, 191, 234]
  },
  {
    name: '交通支出',
    type: 'line',
    //stack: 'Total',
    data: [150, 232, 201, 154]
  },
  {
    name: '教育支出',
    type: 'line',
    //stack: 'Total',
    data: [320, 332, 301, 334]
  },
  {
    name: '医疗支出',
    type: 'line',
    //stack: 'Total',
    data: [820, 932, 901, 934]
  }, {
    name: '娱乐支出',
    type: 'line',
    //stack: 'Total',
    data: [820, 932, 901, 934]
  }, {
    name: '其他支出',
    type: 'line',
    //stack: 'Total',
    data: [820, 932, 901, 934]
  }
]


//饼图配置部分
const pieDom = ref() // 使用ref创建虚拟DOM引用，使用时用pieDom.value
var myPieChart: any;
const optionPie = ref({
  legend: {
    top: 'bottom'
  },
  toolbox: {
    show: true,
    feature: {
      mark: { show: true },
      dataView: { show: true, readOnly: false },
      restore: { show: true },
      saveAsImage: { show: true }
    }
  },
  series: [
    {
      name: 'Nightingale Chart',
      type: 'pie',
      radius: [50, 250],
      center: ['50%', '50%'],
      roseType: 'area',
      itemStyle: {
        borderRadius: 8
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
    text: '季度不同支出'
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
</style>

