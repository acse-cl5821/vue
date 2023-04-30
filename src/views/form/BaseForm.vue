<template>
    <div class="app-container">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i> 付款</el-breadcrumb-item>
                <el-breadcrumb-item>充值信息表单</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="form-box">
                <el-form ref="form" :model="form" label-width="80px">
                    <el-form-item label="公司名称">
                        <el-input v-model="form.merchname" placeholder="MerchId"></el-input>
                    </el-form-item>
                    <el-form-item label="分店名称">
                        <el-input v-model="form.branchname" placeholder="BranchId"></el-input>
                    </el-form-item>
                    <el-form-item label="当前余额">
                      <el-input v-model="form.balance" placeholder="Balance" :disabled="true"></el-input>
                    </el-form-item>
                    <!-- <el-form-item label="选择平台">
                        <el-checkbox-group v-model="form.type">
                            <el-checkbox label="HungryPanda" name="hp"></el-checkbox>
                            <el-checkbox label="Deliveroo" name="dr"></el-checkbox>
                            <el-checkbox label="Uber Eats" name="ue"></el-checkbox>
                            <el-checkbox label="Fantuan" name="ft"></el-checkbox>
                        </el-checkbox-group>
                    </el-form-item> -->
                    <el-form-item label="选择平台">
                        <el-select v-model="form.region" placeholder="请选择">
                            <el-option key="HungryPanda" label="HungryPanda" value="HungryPanda"></el-option>
                            <el-option key="Deliveroo" label="Deliveroo" value="Deliveroo"></el-option>
                            <el-option key="UberEats" label="UberEats" value="UberEats"></el-option>
                            <el-option key="Fantuan" label="Fantuan" value="Fantuan"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="续费期限">
                        <el-radio-group v-model="form.resource">
                            <el-radio label="一个月 £15"></el-radio>
                            <el-radio label="六个月 £72 (£12 monthly)"></el-radio>
                            <el-radio label="一年 £120 (£10 monthly)"></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="支付方式">
                        <el-radio-group v-model="form.paytype">
                            <el-radio label="银行卡"></el-radio>
                            <el-radio label="微信支付"></el-radio>
                            <el-radio label="支付宝"></el-radio>
                            <el-radio label="余额支付"></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <!-- <el-form-item label="日期时间">
                        <el-col :span="11">
                            <el-date-picker type="date" placeholder="选择日期" v-model="form.date1" style="width: 100%;"></el-date-picker>
                        </el-col>
                        <el-col class="line" :span="2">-</el-col>
                        <el-col :span="11">
                            <el-time-picker placeholder="选择时间" v-model="form.date2" style="width: 100%;"></el-time-picker>
                        </el-col>
                    </el-form-item> -->
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit">提交</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </div>

    </div>
</template>

<script>
import { login } from '@/api/login';
import { global } from '@/global/global';
    export default {
      name: 'baseform',
      url: "https://api.tgpaypro.com/v2/tgpaybankcard.php?",
      data: function() {
        return {
          form: {
            merchname: sessionStorage.getItem('username'),
            branchname: sessionStorage.getItem('branchname'),
            balance: sessionStorage.getItem('balance'),
            region: '',
            resource: '一个月 £15',
            paytype: '银行卡',
          }
        }
      },
      methods: {
        async onSubmit() {
          if(this.form.region === ''){
            this.$message.error("请选择充值平台");
            return;
          }
          if(this.form.paytype !== '银行卡' && this.form.paytype !== '余额支付'){
            this.$message.error("目前只支持银行卡付款或余额支付");
            return;
          }
          var price = 0.
          var months = 0
          if(this.form.resource == "一个月 £15") {
            price += 15
            months = 1
          }
          else if(this.form.resource == "六个月 £72 (£12 monthly)") {
            price += 72
            months = 6
          }
          else if(this.form.resource == "一年 £120 (£10 monthly)") {
            price += 120
            months = 12
          }
          if(this.form.paytype === '余额支付') {
            if(price > this.form.balance) {
              this.$message.error("余额不足，请先充值余额，或使用其他支付方式");
              return;
            }
            else {
              fetch("http://3.11.136.6:8000/validate/"+this.form.merchname+"/"+this.form.branchname+"/"+this.form.region+"/"+months)
              open("http://3.11.136.6:8000/validate/"+this.form.merchname+"/"+this.form.branchname+"/-"+price, '_self')
              return;
            }
          }
          var rand12nums = ''
          for (let i = 0; i < 12; i++) {
            rand12nums+=Math.round(Math.random()*9)
          }
          var args = "access_type=cashier"+
          "&channel_type=online"+
          "&goods_name=testing_good"+
          "&language=cn"+
          "&merchants_id=202109208001"+
          // "&notify_url=http://3.11.136.6:8000/"+this.form.merchname+"/"+this.form.branchname+"/"+this.form.region+"/"+months+
          "&out_trade_no=202109208001"+rand12nums+
          "&pay_type=bankcard"+
          "&redirect_url=http://3.11.136.6:8000/validate/"+this.form.merchname+"/"+this.form.branchname+"/"+this.form.region+"/"+months+
          // "&redirect_url=http://localhost:9528"+
          "&return_url=http://localhost:9528/"+
          "&terminal_type=web"+
          "&trans_amount="+price+
          "&trans_timeout=10"
          var tk = "f1s8fqxtkagtvodbsknbwuimog5wv58v"
          var sign = CryptoJS.MD5(Buffer.from(args+tk, 'utf-8').toString()).toString()
          this.url = "https://api.tgpaypro.com/v2/tgpaybankcard.php?"+args+"&sign="+sign+"&sign_type=MD5"
          const response = await fetch(this.url)
          const data = await response.text()
          var pay_url = data.slice(data.indexOf("<pay_url>")+9,data.indexOf("</pay_url>"))
          if(this.form.paytype == '银行卡'){
            // console.log(data)
            open(pay_url, '_self')
          }
        }
      }
    }
</script>
