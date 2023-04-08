<template>
	<section class="app-container">
		<!--工具条-->
		<!-- <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters" @submit.native.prevent>
				<el-form-item>
					<el-input v-model="filters.name" placeholder="分店编码"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getUsers">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col> -->

		<!--列表-->
		<el-table :data="users" highlight-current-row style="width: 100%;">
			<!-- <el-table-column type="selection" width="55">
			</el-table-column> -->
			<el-table-column width="50">
			</el-table-column>
      		<el-table-column type="index" width="80">
			</el-table-column>
			<el-table-column prop="name" label="公司编码" width="130">
			</el-table-column>
			<el-table-column prop="addr" label="分店编码" width="130">
			</el-table-column>
			<el-table-column prop="age" label="外卖平台" width="220">
			</el-table-column>
			<el-table-column prop="birth" label="有效期至" width="220">
			</el-table-column>
			<!-- <el-table-column prop="Deliveroo" label="Deliveroo" width="120">
			</el-table-column>
			<el-table-column prop="Fantuan" label="Fantuan" min-width="160">
			</el-table-column> -->
			<el-table-column label="操作" width="150">
				<template slot-scope="scope">
					<el-button size="small" @click="topup">充值</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<!-- <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button> -->
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="20" :total="total" style="float:right;padding: 20px;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<!-- <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别">
					<el-radio-group v-model="editForm.sex">
						<el-radio class="radio" :label=1>男</el-radio>
						<el-radio class="radio" :label=0>女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="editForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="editForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="editForm.addr"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
			 <el-button @click.native="dialogFormVisible=false">取消</el-button>
			  <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">添加</el-button>
        <el-button v-else type="primary" @click="updateData">修改</el-button>
			</div>
		</el-dialog> -->
	</section>
</template>

<script>
import util from '@/utils/table.js';
import {global} from '@/global/global';
import {
  getUserListPage,
  removeUser,
  batchRemoveUser,
  editUser,
  addUser
} from '@/api/userTable';

export default {
  data() {
    return {
	  merchname: sessionStorage.getItem('username'),
      branchname: sessionStorage.getItem('branchname'),
	  response: '',
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogFormVisible: false,
      filters: {
        name: ''
      },
      users: [],
      total: 0,
      page: 1,
      sels: [],
    }
  },
  methods: {
    handleCurrentChange(val) {
      this.page = val
      this.getUsers()
    },
    // 获取用户列表
    getUsers() {
      const para = {
        page: this.page,
        name: this.filters.name
      }
      getUserListPage(para).then(async res => {
		await fetch('http://35.178.194.182:8000/validate/validate', {
			method: 'POST',
			body: JSON.stringify({ "MerchID": this.merchname, "BranchID": this.branchname})
		})
		.then(response => response.json())
		.then(response => {
			this.response = response
			if (response.code == 0){
				if(this.response.HP_Valid)
					sessionStorage.setItem('HP_Valid', this.response.HP_Valid)
				else
					sessionStorage.setItem('HP_Valid', '未开通')
				if(this.response.DR_Valid)
					sessionStorage.setItem('DR_Valid', this.response.DR_Valid)
				else
					sessionStorage.setItem('DR_Valid', '未开通')
				if(this.response.UE_Valid)
					sessionStorage.setItem('UE_Valid', this.response.UE_Valid)
				else
					sessionStorage.setItem('UE_Valid', '未开通')
				if(this.response.FT_Valid)
					sessionStorage.setItem('FT_Valid', this.response.FT_Valid)
				else
					sessionStorage.setItem('FT_Valid', '未开通')
			}
		})
        this.total = 4
        this.users = [
			{id:'1', name:this.merchname, addr: this.branchname, age:'UberEats', birth:sessionStorage.getItem('UE_Valid')},
			{id:'2', name:this.merchname, addr: this.branchname, age:'Deliveroo', birth:sessionStorage.getItem('DR_Valid')},
			{id:'3', name:this.merchname, addr: this.branchname, age:'HungryPanda', birth:sessionStorage.getItem('HP_Valid')},
			{id:'4', name:this.merchname, addr: this.branchname, age:'Fantuan', birth:sessionStorage.getItem('FT_Valid')},
		]
		// this.total = res.data.total
		// this.users = res.data.users
      })
    },
	topup(){
		console.log()
		this.$router.push({ path: '/form/Baseform' })
	},
  },
  mounted() {
    this.getUsers()
  }
}
</script>

<style scoped>

</style>