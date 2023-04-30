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
                  
                  <el-form-item label="充值金额">
                      <el-radio-group v-model="form.resource">
                          <el-radio label="£50"></el-radio>
                          <el-radio label="£100"></el-radio>
                          <el-radio label="£200"></el-radio>
                          <el-radio label="£500"></el-radio>
                          <el-radio label="自定义"></el-radio>
                      </el-radio-group>
                  </el-form-item>
                  <el-form-item label="自定义">
                      <el-input v-if="form.resource=='自定义'" v-model="form.balancetoadd" placeholder="BalanceToAdd"></el-input>
                      <el-input v-else v-model="form.balancetoadd" placeholder="BalanceToAdd" :disabled="true"></el-input>
                  </el-form-item>
                  <el-form-item label="支付方式">
                      <el-radio-group v-model="form.paytype">
                          <el-radio label="银行卡"></el-radio>
                          <el-radio label="微信支付"></el-radio>
                          <el-radio label="支付宝"></el-radio>
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
          resource: '£50',
          paytype: '银行卡',
        }
      }
    },
    methods: {
      async onSubmit() {
        if(this.form.paytype !== '银行卡'){
          this.$message.error("目前只支持银行卡付款");
          return;
        }
        var price = 0.
        if(this.form.resource == "£50") {
          price += 50
        }
        else if(this.form.resource == "£100") {
          price += 100
        }
        else if(this.form.resource == "£200") {
          price += 200
        }
        else if(this.form.resource == "£500") {
          price += 500
        }
        else {
          price += this.form.balancetoadd
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
        "&redirect_url=http://3.11.136.6:8000/validate/"+this.form.merchname+"/"+this.form.branchname+"/"+price+
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

<!-- <template>
    <div class="app-container">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-date"></i> 表单</el-breadcrumb-item>
                <el-breadcrumb-item>图片上传</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="content-title">支持拖拽</div>
            <div class="plugins-tips">
                Element UI自带上传组件。
                访问地址：<a href="http://element.eleme.io/#/zh-CN/component/upload" target="_blank">Element UI Upload</a>
            </div>
            <el-upload
                class="upload-demo"
                drag
                action="/api/posts/"
                multiple>
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
            <div class="content-title">支持裁剪</div>
            <div class="plugins-tips">
                vue-cropperjs：一个封装了 cropperjs 的 Vue 组件。
                访问地址：<a href="https://github.com/Agontuk/vue-cropperjs" target="_blank">vue-cropperjs</a>
            </div>
            <div class="crop-demo">
                <img :src="cropImg" class="pre-img">
                <div class="crop-demo-btn">选择图片
                    <input class="crop-input" type="file" name="image" accept="image/*" @change="setImage"/>
                </div>
            </div>
        
            <el-dialog title="裁剪图片" :visible.sync="dialogVisible" width="30%">
                <vue-cropper ref='cropper' :src="imgSrc" :ready="cropImage" :zoom="cropImage" :cropmove="cropImage" style="width:100%;height:300px;"></vue-cropper>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="cancelCrop">取 消</el-button>
                    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
                </span>
            </el-dialog>
        </div>
    </div>
</template>

<script>
    import VueCropper from 'vue-cropperjs'
export default {
      name: 'upload',
      data: function() {
        return {
          defaultSrc: './static/img/img.jpg',
          fileList: [],
          imgSrc: '',
          cropImg: '',
          dialogVisible: false
        }
      },
      components: {
        VueCropper
      },
      methods: {
        setImage(e) {
          const file = e.target.files[0]
          if (!file.type.includes('image/')) {
            return
          }
          const reader = new FileReader()
          reader.onload = (event) => {
            this.dialogVisible = true
            this.imgSrc = event.target.result
            this.$refs.cropper && this.$refs.cropper.replace(event.target.result)
          }
          reader.readAsDataURL(file)
        },
        cropImage() {
          this.cropImg = this.$refs.cropper.getCroppedCanvas().toDataURL()
        },
        cancelCrop() {
          this.dialogVisible = false
          this.cropImg = this.defaultSrc
        },
        imageuploaded(res) {
          console.log(res)
        },
        handleError() {
          this.$notify.error({
            title: '上传失败',
            message: '图片上传接口上传失败，可更改为自己的服务器接口'
          })
        }
      },
      created() {
        this.cropImg = this.defaultSrc
  }
    }
</script>

<style scoped>
    .content-title{
        font-weight: 400;
        line-height: 50px;
        margin: 10px 0;
        font-size: 22px;
        color: #1f2f3d;
    }
    .pre-img{   
        width: 100px;
        height: 100px;
        background: #f8f8f8;
        border: 1px solid #eee;
        border-radius: 5px;
    }
    .crop-demo{
        display: flex;
        align-items: flex-end;
    }
    .crop-demo-btn{
        position: relative;
        width: 100px;
        height: 40px;
        line-height: 40px;
        padding: 0 20px;
        margin-left: 30px;
        background-color: #409eff;
        color: #fff;
        font-size: 14px;
        border-radius: 4px;
        box-sizing: border-box;
    }
    .crop-input{
        position: absolute;
        width: 100px;
        height: 40px;
        left: 0;
        top: 0;
        opacity: 0;
        cursor: pointer;
    }
</style> -->