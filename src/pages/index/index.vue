<template>
  <div class="container">
    <i-spin v-if="spinShow" custom>加载中...</i-spin>
    <div class="wrap">
      <div class="channel-wrap">
        <i-tabs :current="params.channel_id" class="channel-tabs"
                :key="item.id"
                v-for="item in channels"
                scroll @change="handleChangeScroll(item)">
          <i-tab :key="item.id" :title="item.name"></i-tab>
        </i-tabs>
        <!--渠道：<i-button inline :type="params.channel_id == item.id ? 'primary': ''"
                        :key="item.id"
                        v-for="item in channels" @click="onSelectChannel(item)" size="small" shape="circle">{{item.name}}</i-button>-->
      </div>
      <div class="filters">
        <picker mode="date" :value="date" start="2018-11-20" @change="bindDateChange">
          <view class="picker">
            日期： {{params.dt}}
          </view>
        </picker>

      </div>

    <!--  <mpvue-echarts lazyLoad   canvasId="line" :echarts="echarts" :onInit="handleInit2" ref="lineCharts" />-->
      <div class="chart-wrap">
        <i-panel>
          <view style="padding: 15px;">总量：<i style="color: red;display: inline">{{total}}</i></view>
        </i-panel>
        <mpvue-echarts canvasId="bar" lazyLoad  :echarts="echarts" :onInit="handleInit" ref="columnCharts" />
      </div>

      <div class="tab-wrap">
        <i-tab-bar :current="currentTab" @change="onSelectTab" color="#f759ab">
          <i-tab-bar-item key="homepage" icon="homepage" current-icon="homepage_fill" title="数据"></i-tab-bar-item>
          <i-tab-bar-item key="group" icon="group" current-icon="group_fill" title="我的"></i-tab-bar-item>
          <i-tab-bar-item key="remind" icon="remind" current-icon="remind_fill" count="3" title="通知"></i-tab-bar-item>
        </i-tab-bar>
      </div>


    </div>
  </div>
</template>

<script>
  import * as echarts from 'echarts' // '@/libs/echarts.simple.min'
  import mpvueEcharts from 'mpvue-echarts'
  import { request } from 'wx-promise-request'
  // import moment from 'moment'
  // import WxCharts from '@/libs/we-charts'

  let chart = null
  export default {
    data () {
      return {
        currentTab: 'homepage',
        spinShow: false,
        total: 0,
        date: '',
        channels: [{
          id: 'cdzyyx',
          name: 'Cdzyyx'
        }],
        params: {
          channel_id: 'cdzyyx',
          dt: '',
          r: ''
        },
        position: 'left',
        option: null,
        option2: null,
        echarts
      }
    },
    components: {
      mpvueEcharts
    },
    created () {
      this.initFilter()
      this.getList()
    },
    watch: {
      params: {
        handler: function (val, oldVal) {
          this.getList()
        },
        deep: true
      }
    },
    methods: {
      getToday () {
        let myDate = new Date()
        let myMonth = myDate.getMonth() + 1
        if (myMonth < 10) {
          myMonth = '0' + myMonth // 补齐
        }
        let mydate = myDate.getDate()
        if (myDate.getDate() < 10) {
          mydate = '0' + myDate.getDate() // 补齐
        }
        let today = myDate.getFullYear() + '-' + myMonth + '-' + mydate
        return today
      },
      initFilter () {
        let date = this.getToday()
        // this.date = date
        this.params.dt = date
      },
      handleChangeScroll (item) {
        this.params.channel_id = item.id
      },
      onSelectTab (item) {
        console.log(item)
      },
      bindDateChange (event) {
        this.params.dt = event.mp.detail.value
      },
      getList () {
        this.spinShow = true
        request({
          url: 'https://api.xushiyun.com/data/dau-incr',
          data: this.params,
          header: {
            'content-type': 'application/json'
          }
        })
          .then(res => {
            let data = res.data
            this.spinShow = false
            if (data.code === 0) {
              this.total = data.data.total
              console.log(this.total)
              this.initChart(data.data)
            }
          })
          .catch(error => console.error(error))
      },
      initChart (data) {
        let categories = data['xAxis']
        let columnSeries = {
          name: '分时引入量',
          data: data['series']
        }
        let total = {
          type: 'line',
          name: '总量',
          data: data['all']
        }
        var options = {
          color: ['#37a2da', '#32c5e9', '#67e0e3'],
          tooltip: {
            trigger: 'axis',
            axisPointer: { // 坐标轴指示器，坐标轴触发有效
              type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
            }
          },
          legend: {
            data: []
          },
          grid: {
            left: 20,
            right: 20,
            bottom: 15,
            top: 40,
            containLabel: true
          },
          yAxis: [
            {
              axisTick: { show: false },
              axisLine: {
                lineStyle: {
                  color: '#999'
                }
              },
              axisLabel: {
                color: '#666'
              }
            }
          ],
          series: []
        }
        this.option = Object.assign({}, options, {
          legend: {
            data: ['引入']
          },
          xAxis: [
            {
              type: 'value',
              axisLine: {
                lineStyle: {
                  color: '#999'
                }
              },
              axisLabel: {
                color: '#666'
              }
            }
          ],
          yAxis: [
            {
              type: 'category',
              data: categories
            }
          ],
          series: [
            {
              name: '引入',
              type: 'bar',
              label: {
                normal: {
                  show: true,
                  position: 'inside'
                }
              },
              data: data['series'],
              itemStyle: {
                emphasis: {
                  color: '#37a2da'
                }
              }
            }
          ]
        })
        /* this.option2 = Object.assign({}, {
          legend: {
            data: ['总量']
          },
          grid: {
            containLabel: true
          },
          tooltip: {
            trigger: 'axis'
          },
          xAxis: [
            {
              type: 'category',
              data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
            }
          ],
          series: [
            {
              name: 'A商品',
              type: 'line',
              smooth: true,
              data: [18, 36, 65, 30, 78, 40, 33]
            }
          ]
        }) */
        this.$refs.columnCharts.init()
        // this.$refs.lineCharts.init()
      },
      handleInit (canvas, width, height) {
        let chart = echarts.init(canvas, null, {
          width: width,
          height: 320
        })
        canvas.setChart(chart)
        chart.setOption(this.option)
        return chart
      }
    }
  }
</script>

<style scoped>
  .wrap {
    width: 100%;
    height: 300px;
  }
  .filters {
    border-radius: 10rpx;
    margin-left: 10rpx;
    margin-right: 10rpx;
    margin-top: 15rpx;
    margin-bottom: 15rpx;
    background-color: #ffffff;
    padding: 15rpx;
    font-size: 24rpx;
  }
  .channel-wrap{
    margin-top: 3rpx;
  }
  .chart-wrap{
    border-radius: 10rpx;
    margin-left: 10rpx;
    margin-right: 10rpx;
    height: 820rpx;
    background-color: #ffffff;
  }
  .tab-wrap{
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
  }
</style>