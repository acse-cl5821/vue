<template>
  <div>
    <el-date-picker id="start" type="date" placeholder="选择起始日期" v-model="startdate" :disabled="true"></el-date-picker>
    <el-date-picker id="end" type="date" placeholder="选择终止日期" v-model="enddate"></el-date-picker>
    <el-button type="primary" @click="getData">查询</el-button>
    <p class="warn-content">
      <a>订单数量
      </a>
    </p>
    <ve-heatmap :data="chartData2"></ve-heatmap>
    <p class="warn-content">
      <a>订单金额
      </a>
    </p>
    <ve-heatmap :data="chartData"></ve-heatmap>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        merchname: sessionStorage.getItem('username'),
        branchname: sessionStorage.getItem('branchname'),
        startdate: "",
        enddate: "",
        chartData: {
          columns: ['日期', '时间', '营业额'],
          rows: [
          ]
        },
        chartData2: {
          columns: ['日期', '时间', '单量'],
          rows: [
          ]
        },
      }
    },
    methods: {
      async getData(){
        await this.setStartDate()
        await fetch('http://3.11.136.6:8000/order/heat/'+this.merchname+'/'+this.branchname+'/'+document.getElementById('start').value+'/'+document.getElementById('end').value, {
          method: 'GET'
        })
        .then(response => response.json())
        .then(response => {
          this.chartData.rows = []
          this.chartData2.rows = []
          for(let i=0; i<response.data.length; i++){
            let row = {}
            row['日期'] = response.data[i]['日期'] + "\n" + this.daytostring(new Date(response.data[i]['日期']).getDay())
            row['时间'] = response.data[i]['时间']
            row['营业额'] = response.data[i]['营业额']
            this.chartData.rows.push(row)
            let row2 = {}
            row2['日期'] = response.data2[i]['日期'] + "\n" + this.daytostring(new Date(response.data[i]['日期']).getDay())
            row2['时间'] = response.data2[i]['时间']
            row2['单量'] = response.data2[i]['单量']
            this.chartData2.rows.push(row2)
          }
        })
      },
      daytostring(day) {
        if(day==0) return "Sunday"
        if(day==1) return "Monday"
        if(day==2) return "Tuesday"
        if(day==3) return "Wednesday"
        if(day==4) return "Thursday"
        if(day==5) return "Friday"
        if(day==6) return "Saturday"
      },
      setStartDate() {
        this.startdate = new Date(this.enddate.getTime() - 518400000)
      }
    },
    mounted() {
      this.startdate = new Date(new Date().getTime() - 518400000)
      this.enddate = new Date(this.startdate.getTime() + 518400000)
      this.getData()
    }
  }
</script>