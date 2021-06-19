<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>历史数据</el-breadcrumb-item>
      </el-breadcrumb>
    </div>

    <div class="container">
      <el-table :data="tableData.slice((currentPage-1)*pageSize,currentPage*pageSize)"
        :row-class-name="tableRowClassName" :header-cell-style="headerRowClass" style="width: 100%">

                <el-table-column prop="collectTime" label="采集时间" align="center">
                </el-table-column>

                <el-table-column prop="mcuVoltage" label="mcu电压（V）" align="center">
                </el-table-column>

                <el-table-column prop="batteryVoltage" label="电池电压（V）" align="center">
                </el-table-column>

                <el-table-column prop="stress1" label="stress1" align="center">
                </el-table-column>
                                <el-table-column prop="stress2" label="stress2" align="center">
                </el-table-column>

                                <el-table-column prop="pressure1" label="pressure1" align="center">
                </el-table-column>

                                <el-table-column prop="pressure2" label="pressure2" align="center">
                </el-table-column>

                                <el-table-column prop="current" label="电流" align="center">
                </el-table-column>

                <el-table-column prop="createdAt" label="创建时间" align="center">
                </el-table-column>

                <el-table-column prop="updatedAt" label="更新时间" align="center">
                </el-table-column>
        <!-- <el-table-column prop="count" label="动作次数" align="center">
        </el-table-column>

        <el-table-column prop="battery" label="电池电量（V）" align="center">
        </el-table-column>

        <el-table-column prop="date" label="时间" align="center">
        </el-table-column> -->


      </el-table>

      <div class="block" style="margin-top:10px;">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage"
          :page-sizes="[10, 20, 30]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper"
          :total="tableData.length" prev-text="上一页" next-text="下一页">
        </el-pagination>
      </div>

      <el-card shadow="hover">
        <el-row :gutter="20">
            <div ref="myChart" class="echart"></div>

          <!-- <el-col :span='12'>
            <div ref="myChart" class="echart"></div>
          </el-col> -->
          <!-- <el-col :span='12'>
            <div ref="myChart2" class="echart"></div>
          </el-col> -->
        </el-row>
      </el-card>

      <div class="title">设备号：{{deviceId}} &nbsp;， 位置号：{{locationInfo}}</div>

      <el-row :gutter='20' type="flex" justify="center" style="margin-top:20px">
        <el-col :span='6'>
          <el-input placeholder="deviceId" suffix-icon="el-icon-cherry" v-model="selectDevice"></el-input>
        </el-col>
        <el-col :span='2'>
          <el-button type="primary" icon="el-icon-search search" @click="search">搜索</el-button>
        </el-col>
      </el-row>
      <br>
      <!-- <button @click="drawChart">test</button> -->
    </div>
    <!-- <button @click="searchDetail">test</button> -->
  </div>
</template>
<script>
  import axios from 'axios';
  let echarts = require('echarts/lib/echarts')
  require('echarts/lib/chart/line')
  require('echarts/lib/chart/bar')
  // 以下的组件按需引入
  require('echarts/lib/component/tooltip') // tooltip组件
  require('echarts/lib/component/title') //  title组件
  require('echarts/lib/component/legend') // legend组件

  export default {
    data() {
      return {
        tableData: [],
        pageSize: 10,
        currentPage: 1,
        selectDevice: '',
        deviceId: '',
        locationInfo: '',
      }
    },
    computed: {},
    methods: {
      tableRowClassName({
        row,
        rowIndex
      }) {
        if (row);
        if (rowIndex % 2 == 1) {
          return 'success-row';
        } else {
          return '';
        }
      },
      headerRowClass({
        row,
        column,
        rowIndex,
        columnIndex
      }) {
        //表头的背景颜色
        if (row || column || columnIndex);

        if (rowIndex == 0) {
          return 'background:oldlace'
        }
      },
      search() {
        if (this.selectDevice == '') {
          return
        }
        this.deviceId = this.selectDevice
        axios.post('/api/findEquipByDevice',{
          deviceId:this.deviceId
        }).then(res=>{
          console.log(res.data)
          this.locationInfo = res.data.data[0].locationInfo
        })

      axios.post('/api/findDataByDeviceId',{
        deviceId:this.selectDevice
      }).then(response=>{
        // console.log()
        let data = response.data.data
        for(let i=0;i<data.length;i++){
          // data[i].collectTime = new Date(new Date(data[i].collectTime).getTime()+30)
          let time = new Date(new Date(data[i].collectTime).getTime()+30*i);

          data[i].collectTime = this.getFormatDate(time);
        }
        this.tableData = data
      }).then(()=>{
        this.drawChart()
      }).catch((err) => {
          console.log(err)
          this.$message.error("设备号输入错误")
        })
      },
      handleSizeChange(val) {
        // console.log(`每页 ${val} 条`);
        this.pageSize = val;
      },
      handleCurrentChange(val) {
        // console.log(`当前页: ${val}`);
        this.currentPage = val;
      },
     getFormatDate(date) {
        var seperator1 = "/";
        var seperator2 = ":";
        var month = date.getMonth() + 1;
        var strDate = date.getDate();
        
        if (month >= 1 && month <= 9) {
            month = "0" + month;
        }
        if (strDate >= 0 && strDate <= 9) {
            strDate = "0" + strDate;
        }
        var currentdate = date.getFullYear() + seperator1 + month + seperator1 + strDate
                + " " + date.getHours() + seperator2 + date.getMinutes() 
                + seperator2 + date.getSeconds();

        let mil = date.getTime()-new Date(currentdate).getTime()
        
        return currentdate + '.' + mil;
    },
      drawChart() {
        let collectTimeArray = [],
          stress1Array = [],
          stress2Array = [],
          pressure1Array = [],
          pressure2Array = [],
          batteryVoltageArray = []

        for (let i = 0; i < this.tableData.length; i++) {
          collectTimeArray.push(this.tableData[i].collectTime);
          stress1Array.push(this.tableData[i].stress1);
          stress2Array.push(this.tableData[i].stress2);
          pressure1Array.push(this.tableData[i].pressure1);
          pressure2Array.push(this.tableData[i].pressure2);
          batteryVoltageArray.push(this.tableData[i].batteryVoltage);
        }

        // let option2 = {
        //   title: {
        //     text: "电池电量"
        //   },
        //   tooltip: {
        //     trigger: 'axis',
        //     axisPointer: {
        //       type: 'shadow'
        //     }
        //   },
        //   xAxis: {
        //     type: 'category',
        //     data: collectTimeArray
        //   },
        //   yAxis: {
        //     type: 'value'
        //   },
        //   grid: {
        //     left: '3%',
        //     right: '4%',
        //     bottom: '3%',
        //     containLabel: true
        //   },
        //   series: [{
        //     name: '电池电量',
        //     data: batteryVoltageArray,
        //     type: 'bar',
        //     showBackground: true,
        //     backgroundStyle: {
        //       color: 'rgba(220, 220, 220, 0.8)'
        //     }
        //   }]
        // }

        let option = {
          title: {
            text: '采集的信息'
          },
          tooltip: {
            trigger: 'axis'
          },
          legend: {
            data: ['Stress1', 'Stress2','Pressure1','Pressure2']
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
            data: collectTimeArray
          },
          yAxis: {
            type: 'value'
          },
          series: [{
              name: 'Stress1',
              type: 'line',
              data: stress1Array
            },
            {
              name: 'Stress2',
              type: 'line',
              data: stress2Array
            },
            {
              name: 'Pressure1',
              type: 'line',
              data: pressure1Array
            },
            {
              name: 'Pressure2',
              type: 'line',
              data: pressure2Array
            },
          ]
        };
        let mychart = echarts.init(this.$refs.myChart);
        // mychart.clear();
        mychart.setOption(option);
        // console.log(option)

        // let mychart2 = echarts.init(this.$refs.myChart2);
        // mychart2.setOption(option2);
      }
      
    },
    mounted() {
      this.selectDevice = this.$store.state.device;
      // console.log(this.selectDevice )
      this.deviceId = this.selectDevice
      this.search()
      // if (this.selectDevice == '') {
      //   axios.get('/api/findEquip').then(response => {
      //     let res = response.data;
      //     this.selectDevice = res.data[0].deviceId;
      //     this.search();
      //   })
      // } else {
      //   this.search();
      // }
    }
  }
</script>

<style lang="less" scoped>
  .title {
    font-weight: 400;
    line-height: 50px;
    margin: 10px 0;
    font-size: 18px;
    color: #1f2f3d;
  }

  .echart {
    width: 100%;
    height: 500px;
  }

  .legend {
    float: right;
    margin-right: 10px;
    line-height: 100%;

    span {
      margin-right: 10px;
    }
  }
</style>