<template>
  <div>
    <el-date-picker id="start" type="date" placeholder="选择起始日期" v-model="startdate" :disabled="true"></el-date-picker>
    <el-date-picker id="end" type="date" placeholder="选择终止日期" v-model="enddate"></el-date-picker>
    <el-button type="primary" @click="getData">查询</el-button>
    <p class="warn-content">
      <a>订单数量
      </a>
    </p>
    <ve-histogram :data="chartData1"></ve-histogram>
    <p class="warn-content">
      <a>订单金额
      </a>
    </p>
    <ve-histogram :data="chartData2"></ve-histogram>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      merchname: sessionStorage.getItem('username'),
      branchname: sessionStorage.getItem('branchname'),
      startdate: new Date(new Date() - ((new Date().getDate() - 1) * 24 * 3600 * 1000)),
      enddate: new Date(),
      chartData1: {
        columns: ['日期', '总单量', 'HungryPanda', 'Deliveroo', 'UberEats', 'Fantuan'],
        rows: []
      },
      chartData2: {
        columns: ['日期', '总金额', 'HungryPanda', 'Deliveroo', 'UberEats', 'Fantuan'],
        rows: []
      }
    }
  },
  methods: {
    async getData(){
      await this.setDates()
      await fetch('http://3.11.136.6:8000/order/num/'+this.merchname+'/'+this.branchname+'/'+document.getElementById('start').value+'/'+document.getElementById('end').value, {
			  method: 'GET'
		  })
		  .then(response => response.json())
		  .then(response => {
        this.chartData1.rows = response.data
      })
      await fetch('http://3.11.136.6:8000/order/price/'+this.merchname+'/'+this.branchname+'/'+document.getElementById('start').value+'/'+document.getElementById('end').value, {
			  method: 'GET'
		  })
		  .then(response => response.json())
		  .then(response => {
        this.chartData2.rows = response.data
      })
    },
    setDates(){
      this.startdate = new Date(this.enddate.getYear()+1900,this.enddate.getMonth(),1)
      this.enddate = new Date(this.enddate.getYear()+1900,this.enddate.getMonth()+1,0)
      if(this.enddate.getMonth() == new Date().getMonth()){
        this.enddate = new Date()
      }
    }
  },
  mounted() {
    this.getData()
  }
}
</script>