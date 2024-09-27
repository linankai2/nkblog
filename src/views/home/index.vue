<template>
  <div class="FlipClock">
    <Flipper ref="flipperHour" />
    <em>:</em>
    <Flipper ref="flipperMinute" />
    <em>:</em>
    <Flipper ref="flipperSecond" />
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Flipper from './components/Flipper.vue'
const timer = ref<any>(null)
const flipObjs = ref<any[]>([])
const flipperHour = ref()
const flipperMinute = ref()
const flipperSecond = ref()
// 初始化数字
function init() {
  let now = new Date()
  let nowTimeStr = formatDate(new Date(now.getTime()), 'hh-ii-ss')
  nowTimeStr = nowTimeStr.split('-')
  for (let i = 0; i < flipObjs.value.length; i++) {
    flipObjs.value[i].value.setFront(nowTimeStr[i])
  }
}
// 开始计时
function run() {
  timer.value = setInterval(() => {
    // 获取当前时间
    let now = new Date()
    let nowTimeStr = formatDate(new Date(now.getTime() - 1000), 'hh-ii-ss')
    nowTimeStr = nowTimeStr.split('-')
    let nextTimeStr = formatDate(now, 'hh-ii-ss')
    nextTimeStr = nextTimeStr.split('-')
    for (let i = 0; i < flipObjs.value.length; i++) {
      if (nowTimeStr[i] === nextTimeStr[i]) {
        continue
      }
      flipObjs.value[i].value.flipDown(nowTimeStr[i], nextTimeStr[i])
    }
  }, 1000)
}
// 正则格式化日期
function formatDate(date, dateFormat) {
  /* 单独格式化年份，根据y的字符数量输出年份
       * 例如：yyyy => 2019
              yy => 19
              y => 9
       */
  if (/(y+)/.test(dateFormat)) {
    dateFormat = dateFormat.replace(
      RegExp.$1,
      (date.getFullYear() + '').substr(4 - RegExp.$1.length)
    )
  }
  // 格式化月、日、时、分、秒
  let o = {
    'm+': date.getMonth() + 1,
    'd+': date.getDate(),
    'h+': date.getHours(),
    'i+': date.getMinutes(),
    's+': date.getSeconds()
  }
  for (let k in o) {
    if (new RegExp(`(${k})`).test(dateFormat)) {
      // 取出对应的值
      let str = o[k] + ''
      /* 根据设置的格式，输出对应的字符
       * 例如: 早上8时，hh => 08，h => 8
       * 但是，当数字>=10时，无论格式为一位还是多位，不做截取，这是与年份格式化不一致的地方
       * 例如: 下午15时，hh => 15, h => 15
       */
      dateFormat = dateFormat.replace(RegExp.$1, RegExp.$1.length === 1 ? str : padLeftZero(str))
    }
  }
  return dateFormat
}
// 日期时间补零
function padLeftZero(str) {
  return str.toString().padStart(2, '0')
}
onMounted(() => {
  flipObjs.value = [flipperHour, flipperMinute, flipperSecond]
  init()
  run()
})
</script>

<style lang="scss" scoped>
.FlipClock {
  text-align: center;
}
.FlipClock .M-Flipper {
  margin: 0 3px;
}
.FlipClock em {
  display: inline-block;
  line-height: 102px;
  font-size: 66px;
  font-style: normal;
  vertical-align: top;
}
</style>
