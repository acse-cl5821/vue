<template>
  <div>
    <el-date-picker id="start" type="date" placeholder="选择起始日期" v-model="startdate"></el-date-picker>
    <el-date-picker id="end" type="date" placeholder="选择终止日期" v-model="enddate" :disabled="true"></el-date-picker>
    <el-button type="primary" @click="getData">查询</el-button>
    <p class="warn-content">
      <a>订单数量
      </a>
    </p>
    <ve-pie :data="chartData1"></ve-pie>
    <p class="warn-content">
      <a>订单金额
      </a>
    </p>
    <ve-pie :data="chartData2"></ve-pie>
  </div>
</template>

<script>
export default {
  data: function(){
    return {
      merchname: sessionStorage.getItem('username'),
      branchname: sessionStorage.getItem('branchname'),
      startdate: new Date(new Date().getTime() - 518400000),
      enddate: new Date(),
      chartData1: {
        columns: ['日期', '单量'],
        rows: []
      },
      chartData2: {
        columns: ['日期', '金额'],
        rows: []
      }
    }
  },
  methods: {
    async getData(){
      this.enddate = new Date(this.startdate.getTime() + 518400000)
      await fetch('http://3.11.136.6:8000/order/num/'+this.merchname+'/'+this.branchname+'/'+document.getElementById('start').value+'/'+document.getElementById('end').value, {
        method: 'GET'
      })
      .then(response => response.json())
      .then(response => {
        this.chartData1.rows = []
        for(let i=0; i<response.data.length; i++){
          let row = {}
          row['日期'] = this.daytostring(new Date(response.data[i]['日期']).getDay())
          row['单量'] = response.data[i]['总单量']
          this.chartData1.rows.push(row)
        }
      })
      await fetch('http://3.11.136.6:8000/order/price/'+this.merchname+'/'+this.branchname+'/'+document.getElementById('start').value+'/'+document.getElementById('end').value, {
        method: 'GET'
      })
      .then(response => response.json())
      .then(response => {
        console.log(response)
        this.chartData2.rows = []
        for(let i=0; i<response.data.length; i++){
          let row = {}
          row['日期'] = this.daytostring(new Date(response.data[i]['日期']).getDay())
          row['金额'] = response.data[i]['总金额']
          this.chartData2.rows.push(row)
        }
        console.log(this.chartData2.rows)
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
    }
  },
  mounted() {
    this.startdate = new Date(new Date().getTime() - 518400000)
    this.enddate = new Date(this.startdate.getTime() + 518400000)
    this.getData()
  }
}
</script>