<template>
  <div class="addBrand-container">
    <div class="container">
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="180px" v-loading="loading">
        <el-form-item label="账号" prop="username">
          <el-input v-model="ruleForm.username"></el-input>
        </el-form-item>
        <el-form-item label="员工姓名" prop="name">
          <el-input v-model="ruleForm.name"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="ruleForm.phone"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="sex">
            <el-radio v-model="ruleForm.sex" label="1">男</el-radio>
            <el-radio v-model="ruleForm.sex" label="2">女</el-radio>
        </el-form-item>
        <el-form-item label="身份证号" prop="idNumber">
          <el-input v-model="ruleForm.idNumber"></el-input>
        </el-form-item>
        <div class="subBox">
          <el-button type="primary" @click="submitForm('ruleForm',false)">保存</el-button>
          <el-button 
            v-if="this.optType === 'add'" 
            type="primary" 
            @click="submitForm('ruleForm',true)">保存并继续添加员工
          </el-button>
          <el-button @click="() => this.$router.push('/employee')">返回</el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script lang="ts">
import {addEmployee, getEmployeeById, updateEmployee} from '@/api/employee'
export default {
  data(){
    return{
      loading: true,
      optType: 'add',
      ruleForm: {
        username: '',
        name: '',
        phone: '',
        sex: '1',
        idNumber: ''
      },
      rules: {
        username: [{required:true, message:'请输入员工账号',trigger:'blur'}],
        name: [{required:true, message:'请输入员工姓名',trigger:'blur'}],
        phone: [{required:true, trigger:'blur', 
          // 校验手机号的常用方法
          validator:(rule,value,callback)=>{
            // 手机号的正则校验
            if (value === '' || (!/^1(3|4|5|6|7|8)\d{9}$/.test(value))) {
              // 回调弹出提示
              callback(new Error('请输入正确的手机号！'))
            } else {
              callback()
            }
        }}],
        idNumber: [{required:true, trigger:'blur',
          validator:(rule,value,callback)=>{
            // 身份证号的正则校验
            if (value === '' || (!/(^\d{15}$)|(^\d{18}$)|(^\d{17}(X|x)$)/.test(value))) {
              // 回调弹出提示
              callback(new Error('请输入正确的身份证号！'))
            } else {
              callback()
            }
        }}]
      }
    }
  },
  created(){
    this.optType = this.$route.query.id ? 'update' : 'add'
    if (this.optType === 'update'){
      getEmployeeById(this.$route.query.id).then(res =>{
        if (res.data.code === 1){
          this.ruleForm = res.data.data
          this.loading = false
        }
      })
    } else {
      this.loading = false
    }
  },
  methods:{
    submitForm(formName, isContinue){
      // 进行表单校验,确保没有红色提示
      this.$refs[formName].validate((valid) => {
        if (valid) {
          // 表单校验通过，发送ajax请求
          if (this.optType === 'add'){
            addEmployee(this.ruleForm).then((res)=>{
              if (res.data.code === 1){
                this.$message.success('员工添加成功')
                if (isContinue) {
                  this.ruleForm = {
                    username: '',
                    name: '',
                    phone: '',
                    sex: '1',
                    idNumber: ''
                  }
                } else {
                  this.$router.push('/employee')
                }
              } else {
                this.$message.error(res.data.msg)
              }
            })
          } else {
            updateEmployee(this.ruleForm).then((res) => {
              if (res.data.code === 1){
                this.$message.success('员工修改成功')
                this.$router.push('/employee')
              }
            })
          }
        } else {
          this.$message.error('请按照提示修改信息')
        }
      })
    }
  }

}
</script>

<style lang="scss" scoped>
.addBrand {
  &-container {
    margin: 30px;
    margin-top: 30px;
    .HeadLable {
      background-color: transparent;
      margin-bottom: 0px;
      padding-left: 0px;
    }
    .container {
      position: relative;
      z-index: 1;
      background: #fff;
      padding: 30px;
      border-radius: 4px;
      // min-height: 500px;
      .subBox {
        padding-top: 30px;
        text-align: center;
        border-top: solid 1px $gray-5;
      }
    }
    .idNumber {
      margin-bottom: 39px;
    }

    .el-form-item {
      margin-bottom: 29px;
    }
    .el-input {
      width: 293px;
    }
  }
}
</style>
