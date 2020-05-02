<template>
  <div class="hello">
    <el-container>
      <el-header style="background: #5CACEE;">

        <div class="header_top">
          <span style="width: 90%;color: #FFFFFF;">服务器监控系统</span>
          <img style="width: 2rem; height: 2rem;" src="../assets/logo.png" />
        </div>
      </el-header>
      <el-main>
        <el-card class="box-card">
            <div slot="header" class="clearfix">
              <div>
                <span>客户端：{{engkey}}</span>
                <el-input class="emailto" v-model="input" placeholder="要更改的预警邮箱地址"></el-input>
                <el-button @click="updateEmail()">更改邮箱</el-button>
                <el-button @click="mokeEmial()">模拟预警</el-button>
              </div>

            </div>
            <div class="card_body">

              <div id="myChart" :style="{ height: '300px'}"></div>
              <div id="myChartMem" :style="{ height: '300px'}"></div>
              <div id="myChartDisk" :style="{ height: '500px'}"></div>
              <div>
                <el-row>
                  <span>模拟发送简单指令：</span>
                  <el-button @click="sendDirect('mk')">创建</el-button>
                  <el-button @click="sendDirect('cp')" type="primary">拷贝</el-button>
                  <el-button @click="sendDirect('mv')" type="success">移动</el-button>
                  <el-button @click="sendDirect('rm')" type="danger">删除</el-button>
                </el-row>
              </div>
            </div>
          </el-card>

      </el-main>
   </el-container>
   </div>
</template>

<script>
  import axios from 'axios'
  import echarts from 'echarts'

  export default {
    name: 'Infos',
    data() {
      return {
        showNum: false, //控制图表上方是否显示数字
        direct: '',
        input: '',
        engkey: "",
        datax: [],
        datay: [],
        dataDiskIng: 0,
        dataDiskFree: 100,
        datayMem: [],
        engints: [],
        percent1: 30,
        percent2: 70,
        year: "2018-10-20",
        time: "10:05:03",
        myChartCpu: null,
        myChartMem: null,
        myChartDisk: null,
        timer: null
      }
    },
    mounted() {
      this.drawLine();
    },
    created() {
      let id = this.$route.query.engkey;
      this.engkey = id;
      this.getDataY(50);
      let self = this;
      axios.post("/api/email/get?engkey=" + this.engkey)
        .then(function(response) {
          self.input = response.data.emailTo
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    beforeDestroy() {
      clearInterval(this.timer)
    },
    methods: {
      sendDirect(direct) {
        axios.post("/api/engint/send?id=" + this.engkey + "&direct=" + direct)
          .then(function(response) {
            console.log(response)
          }).catch()
      },
      mokeEmial() {
        axios.post("/api/email/send?id=" + this.engkey + "&email=" + this.input)
          .then(function(response) {
            console.log(response)
          }).catch()
      },
      updateEmail() {
        axios.post("/api/email/update?id=" + this.engkey + "&email=" + this.input)
          .then(function(response) {
            console.log(response)
          }).catch()

      },
      getDataY(size) {
        let self = this;
        axios.post("/api/engint/getOneEngint?engkey=" + this.engkey + "&size=" + size)
          .then(function(response) {
            self.engints = response.data;

            for (let item in self.engints) {
              let number = self.engints.length - item - 1;
              self.datax.push(self.engints[item].createTime);
              self.datay.push(100 - self.engints[item].cpuing.substring(0, 2));
              let memUseable = self.engints[item].memorying.indexOf(" ");
              let memAll = (self.engints[item].memoryall).indexOf(" ");
              let fixMemIng = self.engints[item].memorying.substring(0, memUseable);
              if (fixMemIng > 500) {
                fixMemIng = 1
              }
              let mem = fixMemIng / self.engints[item].memoryall.substring(0, memAll);
              self.datayMem.push((100 - mem * 100 + "").substring(0, 4))


              let diskIng = self.engints[item].disking.indexOf(" ");
              let diskAll = (self.engints[item].diskall).indexOf(" ");
              self.dataDiskFree = self.engints[item].disking.substring(0, diskIng);
              let dataDiskAll = self.engints[item].diskall.substring(0, diskAll);
              if (Number(self.dataDiskFree) > Number(dataDiskAll)) {
                dataDiskAll = dataDiskAll * 1024
              }
              self.dataDiskIng = (dataDiskAll - self.dataDiskFree).toFixed(2)

            }
          })
          .catch(function(error) {
            console.log(error);
          });
      },

      drawLine() {
        let self = this;
        let option = {
          title: {
            text: '动态CPU使用率'
          },
          xAxis: {
            type: 'category',
            boundaryGap: false,
            data: self.datax,

          },
          yAxis: {
            type: 'value',
            max: 100
          },
          series: [{
            itemStyle: {
              color: '#00ff00'
            },
            label: {
              normal: { //展示数字和位置
                show: self.showNum,
                position: 'top'
              }
            },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: '#00ff00'
              }, {
                offset: 1,
                color: '#ffe'
              }])
            },
            data: self.datay,
            type: 'line',
          }]
        };

        let optionMem = {

          title: {
            text: '动态内存使用率'
          },
          xAxis: {
            type: 'category',
            boundaryGap: false,
            data: self.datax,

          },
          yAxis: {
            type: 'value',
            max: 100,
            min: 0

          },
          series: [{
            itemStyle: {
              color: '#00ff00'
            },
            label: {
              normal: { //展示数字 ，及展示位置
                show: self.showNum,
                position: 'top'
              }
            },
            areaStyle: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: '#00ff00'
              }, {
                offset: 1,
                color: '#ffe'
              }])
            },
            data: self.datayMem,
            type: 'line',
          }]
        };

        let optionDisk = {
          title: {
            text: '客户端磁盘状况',
            left: 'center'
          },
          legend: {
            orient: 'vertical',
            left: 'left',
            data: ['使用总量', '空闲总量']
          },
          series: [{
            label: {
              normal: { //展示数字 ，及展示位置
                formatter: ' {b}：{c} 百分比：{d}% ',
                backgroundColor: '#eee',
                borderColor: '#aaa',
                borderWidth: 1,
                padding: [7, 7],
                borderRadius: 4,
              }
            },
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [{
                value: self.dataDiskIng,
                name: '使用总量'
              },
              {
                value: self.dataDiskFree,
                name: '空闲总量'
              }
            ],

          }]
        };


        self.myChartCpu = echarts.init(document.getElementById('myChart'));
        self.myChartMem = echarts.init(document.getElementById('myChartMem'));
        self.myChartDisk = echarts.init(document.getElementById('myChartDisk'));
        self.myChartCpu.setOption(option);
        self.myChartMem.setOption(optionMem);
        self.myChartDisk.setOption(optionDisk);
        self.timer = setInterval(function() {
          self.getDataY(1);
          self.myChartCpu.setOption({
            series: [{
              data: self.datay
            }],
            xAxis: {
              type: 'category',
              boundaryGap: false,
              data: self.datax,
            },
          });
          self.myChartDisk.setOption({
            series: [{
              data: [{
                  value: self.dataDiskIng,
                  name: '使用总量'
                },
                {
                  value: self.dataDiskFree,
                  name: '空闲总量'
                }
              ],
            }]

          });
          self.myChartMem.setOption({
            series: [{
              data: self.datayMem
            }],
            xAxis: {
              type: 'category',
              boundaryGap: false,
              data: self.datax,
            },
          });

        }, 2000);
      }
    }
  }
</script>

<style>
  .emailto {
    margin-left: 20px;
    width: 20%;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both
  }

  .card_body {}

  .box-card {
    width: 100%;
  }
</style>
