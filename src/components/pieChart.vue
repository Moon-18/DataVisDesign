<template>
  <div ref="pieDom" style="width: 500px; height: 500px;display:inline-block"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
//  按需引入 echarts
import * as echarts from "echarts";
const pieDom = ref() // 使用ref创建虚拟DOM引用，使用时用pieDom.value
const props = defineProps<{
  dataList: any
}>()
watch(() => { return props.dataList.value }, () => {
  console.log("父组件更新了");

})
onMounted(
  () => {
    initPie()
  }
)
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
      data: props.dataList
    }
  ]
})
function initPie() {
  // 基于准备好的dom，初始化echarts实例
  var myChart = echarts.init(pieDom.value);
  // 指定图表的配置项和数据
  // 使用刚指定的配置项和数据显示图表。
  myChart.setOption(optionPie.value);
}
</script>


<style scoped>
/* div {
  display: inline-block;
} */
</style>

