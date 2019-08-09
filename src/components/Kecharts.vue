<template>
  <div class="kline_box">
    <div class="kline">
      <div class="title">
        <p>
          BTC
          <span>3975.86（+2.05%）</span>
        </p>
      </div>
      <div :id="id" class="echarts"></div>
    </div>
    <div class="kbar">
      <div
        :class="{'item': acIndex != index, 'active': acIndex == index}"
        v-for="(item,index) in bar"
        :key="index"
        @click="changeBar(index)"
      >{{ item.name }}</div>
    </div>
  </div>
</template>

<script>
import echarts from 'echarts'
import moment from 'moment'
export default {
  props: {
    id: String,
    newData: Array
  },
  data() {
    return {
      num: 0,
      bar: [
        { name: '分时线', val: 'quotek1' },
        { name: '1分线', val: 'quotek1' },
        { name: '5分线', val: 'quotek5' },
        { name: '15分线', val: 'quotek15' },
        { name: '60分线', val: 'quotek60' },
        { name: '日K', val: 'quotekdD' },
        { name: '月K', val: 'quotekdM' },
        { name: '周K', val: 'quotekdK' }
      ],
      acIndex: 0,
      loading: false,
      kType: String,
      datas: [],
      data0: [],
      activeName: 'a',
      myiner: null,
    }
  },
  mounted() {
    this.getKC()

    this.drawTime()
  },
  methods: {
    // 截取数据
    splitData(rawData) {
      var categoryData = []
      var values = []
      for (var i = 0; i < rawData.length; i++) {
        categoryData.push(rawData[i].splice(0, 1)[0])
        values.push(rawData[i])
      }
      return {
        categoryData: categoryData,
        values: values
      }
    },

    calculateMA(dayCount) {
      var result = []
      for (var i = 0, len = data0.values.length; i < len; i++) {
        if (i < dayCount) {
          result.push('-')
          continue
        }
        var sum = 0
        for (var j = 0; j < dayCount; j++) {
          sum += data0.values[i - j][1]
        }
        result.push(sum / dayCount)
      }
      return result
    },
    
    // 分时折线图Y轴数据
    getYdata(rawData) {
      var yData = []
      for (var i = 0; i < rawData.length; i++){
        yData.push(rawData[i][0])
      }

      return yData
    },

    getKC() {
      this.data0 = this.splitData(this.newData)

      this.datas = this.data0
    },


    // 绘制分时折线图
    drawTime() {
      var that = this
      var myChart = echarts.init(document.getElementById(this.id))
      var arr = this.datas.values
      var yData = this.getYdata(arr)
      var xData = this.datas.categoryData

      var option = {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: 'rgba(255,255,255,1)',
              borderColor: '#ccc',
              borderWidth: '2',
              color: '#000',
              shadowBlur: 0,
              padding: [3, 3],
              fontSize: 13
            },
            crossStyle: {
              type: 'solid'
            }
          },
          showContent: false,
          confine: true
        },
        grid: {
          right: '57px'
        },
        xAxis: {
          type: 'category',
          data: that.data0.categoryData,
          scale1: true,
          splitLine: {
            show: true,
            lineStyle: {
              color: ['#eee'],
              width: 1,
              type: 'solid'
            }
          },
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          }
        },
        yAxis: {
          type: 'value',
          scale: true,
          boundaryGap: false,
          position: 'right',
          splitLine: {
            show: true,
            lineStyle: {
              color: ['#eee'],
              width: 1,
              type: 'solid'
            }
          },
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          }
        },
        dataZoom: [
          {
            type: 'inside',
            start: 50,
            end: 100
          },
          {
            show: false,
            type: 'slider',
            x: '90%',
            start: 50,
            end: 100
          }
        ],
        series: [
          {
            name: '数据模拟',
            type: 'line',
            showSymbol: true,
            symbol: 'none',
            sampling: 'average',
            itemStyle: {
              color: '#108EE9'
            },
            lineStyle: {
              width: 1,
              color: '#108EE9'
            },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                {
                  offset: 0,
                  color: '#A2D3F7'
                },
                {
                  offset: 1,
                  color: '#FAFDFF'
                }
              ])
            },
            markLine: {
              silent: false,
              symbol: ['none', 'circle'],
              data: [
                {
                  yAxis: yData[yData.length - 1]
                }
              ],
              label: {
                show: true,
                borderColor: '#8EC9F5',
                backgroundColor: '#fff',
                borderWidth: '1',
                precision: 2,
                fontSize: 12,
                padding: [3, 3]
              }
            },
            data: yData
          }
        ]
      }

      myChart.setOption(option)

      this.myiner = setInterval(function() {
        // 获取当前时分
        var axisData = moment().format("HH:mm")
        // 获取当前秒
        var axisData1 = moment().format("ss")

        var mess = parseFloat(Math.random() * 200) + 2200

        if (axisData1 < 59) {
          yData[yData.length - 1] = mess
        } else {
          yData.shift()
          yData.push(mess)
          option.xAxis.data.shift()
          option.xAxis.data.push(axisData)
        }

        myChart.setOption(
          (option = {
            tooltip: {
              trigger: 'axis',
              axisPointer: {
                type: 'cross',
                label: {
                  backgroundColor: 'rgba(255,255,255,1)',
                  borderColor: '#ccc',
                  borderWidth: '2',
                  color: '#000',
                  shadowBlur: 0,
                  padding: [3, 3],
                  fontSize: 13
                },
                crossStyle: {
                  type: 'dashed'
                }
              },
              showContent: false,
              confine: true
            },
            grid: {
              right: '57px'
            },
            xAxis: {
              type: 'category',
              data: that.data0.categoryData,
              scale1: true,
              splitLine: {
                show: true,
                lineStyle: {
                  color: ['#eee'],
                  width: 1,
                  type: 'solid'
                }
              },
              axisLine: {
                show: false
              },
              axisTick: {
                show: false
              }
            },
            yAxis: {
              type: 'value',
              scale: true,
              boundaryGap: false,
              position: 'right',
              splitLine: {
                show: true,
                lineStyle: {
                  color: ['#eee'],
                  width: 1,
                  type: 'solid'
                }
              },
              axisLine: {
                show: false
              },
              axisTick: {
                show: false
              }
            },
            dataZoom: [
              {
                type: 'inside',
                start: 50,
                end: 100
              },
              {
                show: false,
                type: 'slider',
                x: '90%',
                start: 50,
                end: 100
              }
            ],
            series: [
              {
                name: '数据模拟',
                type: 'line',
                showSymbol: true,
                symbol: 'none',
                sampling: 'average',
                itemStyle: {
                  color: '#108EE9'
                },
                lineStyle: {
                  width: 1,
                  color: '#108EE9'
                },
                areaStyle: {
                  color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                    {
                      offset: 0,
                      color: '#A2D3F7'
                    },
                    {
                      offset: 1,
                      color: '#FAFDFF'
                    }
                  ])
                },
                markLine: {
                  silent: false,
                  symbol: ['none', 'circle'],
                  data: [
                    {
                      yAxis: yData[yData.length - 1]
                    }
                  ],
                  label: {
                    show: true,
                    borderColor: '#8EC9F5',
                    backgroundColor: '#fff',
                    borderWidth: '1',
                    precision: 2,
                    fontSize: 12,
                    padding: [3, 3]
                  }
                },
                data: yData
              }
            ]
          })
        )
      }, 1000)
    },

    // 绘制K线图
    drawKline() {
      var that = this
      var myChart = echarts.init(document.getElementById(this.id))

      var upColor = '#6CC47C'
      var upBorderColor = '#6CC47C'
      var downColor = '#FD8787'
      var downBorderColor = '#FD8787'

      var option = {
        tooltip: {
          triggerOn: 'mousemove|click',
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          formatter: function(params) {
            return (
              '时间 <span style="display: inline-block; width: 3px"></span>' +
              '19/05/10 <span style="display: inline-block; width: 3px"></span>' +
              params[0].name +
              '<br>' +
              '价格' +
              '<span style="display: inline-block; width: 58px;"></span>' +
              params[0].value[1] +
              '<br>' +
              '开' +
              '<span style="display: inline-block; width: 70px;"></span>' +
              params[0].value[1] +
              '<br>' +
              '高' +
              '<span style="display: inline-block; width: 70px;"></span>' +
              params[0].value[2] +
              '<br>' +
              '低' +
              '<span style="display: inline-block; width: 70px;"></span>' +
              params[0].value[3] +
              '<br>' +
              '收' +
              '<span style="display: inline-block; width: 70px;"></span>' +
              params[0].value[4] +
              '<br>' +
              '涨跌额' +
              '<span style="display: inline-block; margin-left: 48px; color: #FD8787">' +
              params[0].value[5] +
              '</span><br>' +
              '涨跌幅' +
              '<span style="display: inline-block; margin-left: 42px; color: #FD8787">' +
              params[0].value[6] +
              '</span>'
            )
          },
          showContent: true,
          backgroundColor: 'rgba(255,255,255,1)',
          borderWidth: 1,
          borderColor: 'rgba(221,221,221,1)',
          textStyle: {
            color: '#000'
          },
          padding: [5, 5, 6, 13]
        },
        xAxis: {
          type: 'category',
          data: this.datas.categoryData,
          scale: true,
          boundaryGap: false,
          axisLine: { onZero: false },
          splitLine: { show: false },
          splitNumber: 20,
          min: 'dataMin',
          max: 'dataMax',
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          },
          splitLine: {
            show: true,
            lineStyle: {
              color: ['#eee'],
              width: 1,
              type: 'solid'
            }
          },
          axisPointer: {
            label: {
              show: true,
              padding: [7, 10],
              borderWidth: 1
            }
          }
        },
        yAxis: {
          scale: true,
          position: 'right',
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          },
          splitLine: {
            show: true,
            lineStyle: {
              color: ['#eee'],
              width: 1,
              type: 'solid'
            }
          },
          axisPointer: {
            label: {
              show: true,
              padding: [7, 10],
              borderWidth: 1
            }
          }
        },
        dataZoom: [
          {
            type: 'inside',
            start: 50,
            end: 100
          },
          {
            show: false,
            type: 'slider',
            y: '90%',
            start: 50,
            end: 100
          }
        ],
        series: [
          {
            name: '日K',
            type: 'candlestick',
            data: this.datas.values,
            itemStyle: {
              normal: {
                color: upColor,
                color0: downColor,
                borderColor: upBorderColor,
                borderColor0: downBorderColor
              }
            }
          }
        ]
      }

      myChart.setOption(option)
    },

    changeBar(val) {
      this.loading = false
      this.acIndex = val
      if (this.myiner != null) {
        clearInterval(this.myiner)
        this.myiner = null
      }

      switch (val) {
        case 0:
          this.drawTime()
          break
        case 1:
          this.drawKline()
          break
        default:
          break
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.kline_box {
  .kline {
    width: 100%;
    position: relative;
    height: 300px;
    position: relative;

    .title {
      font-size: 20px;
      font-weight: 500;
      color: rgba(51, 51, 51, 1);
      line-height: 24px;
      position: absolute;
      top: 0;
      left: 31px;

      span {
        color: #fd8787;
        font-weight: normal;
      }
    }

    .echarts {
      width: 100%;
      height: 300px;
      position: absolute;
      top: 0;
      left: -15px;
    }
  }

  .kbar {
    width: 100%;
    height: 50px;
    line-height: 50px;
    display: flex;
    justify-content: space-around;
    border-bottom: 3px solid #f9f9f9;

    div {
      float: left;
    }
  }

  .active {
    color: #108ee9;
    border-bottom: 2px solid #108ee9;
  }
}
</style>
