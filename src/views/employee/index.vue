<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right: 5px;">员工姓名：</label>
        <el-input v-model="name" placeholder="请输入员工姓名" style="width: 15%;" />
        <el-button type="primary" style="margin-left: 25px;" @click="pageQuery" class="normal-btn continue">查询</el-button>
        <el-button type="primary" style="float:right" @click="() => this.$router.push('/employee/add')">+ 添加员工</el-button>
      </div>
      <el-table
        v-loading="loading"
        :data="records"
        stripe
        style="width: 100%" class="tableBox">
        <el-table-column
          prop="name"
          label="员工姓名"
          width="180">
        </el-table-column>
        <el-table-column
          prop="username"
          label="账号"
          width="180">
        </el-table-column>
        <el-table-column
          prop="phone"
          label="手机号">
        </el-table-column>
        <el-table-column
          prop="status"
          label="账号状态">
          <template slot-scope="scope">
            <div
              class="tableColumn-status"
              :class="{ 'stop-use': String(scope.row.status) === '0' }">
            {{ scope.row.status === 0 ? '禁用' : '启用'}}
            </div>
          </template>
        </el-table-column>
        <el-table-column
          prop="updateTime"
          label="最后操作时间">
        </el-table-column>
        <el-table-column
          label="操作" align="center"> 
          <template slot-scope="scope">
            <el-button type="text" size="small" class="blueBug" @click="editEmployee(scope.row.id)"
            :class="{
                'disabled-text': scope.row.username === 'admin'
              }">修改</el-button>
            <el-button type="text" size="small"
              class="non"
              :class="{
                'disabled-text': scope.row.username === 'admin',
                blueBug: scope.row.status == '0',
                delBut: scope.row.status != '0',
              }"
              @click="handleStatus(scope.row.id, scope.row.status)">
              {{ scope.row.status === 1 ? '禁用' : '启用'}}
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
       class="pageList"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page"
        :page-sizes="[5, 10]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>
  </div>
</template>
<script lang="ts">
import {getEmployeeList, enableOrDisableEmployee} from '@/api/employee'
export default  {
  data() {
    return {
      records: [], // 当前要展示的数据集合
      total: 0,
      page: 1,
      pageSize: 5,
      name: '',
      loading: true
    }
  },
  created(){
    this.pageQuery()
  },
  methods:{
    // 分页查询
    pageQuery(){
      // 准备请求参数
      const params = {
        name: this.name,
        page: this.page,
        pageSize: this.pageSize
      }
      // 发送ajax请求，访问后端服务，获取分页数据
      getEmployeeList(params).then(res => {
        if (res.data.code === 1){
          this.total = res.data.data.total
          this.records = res.data.data.records
          this.loading = false
        }
      }).catch(err => {
        this.$message.error('请求出错了：' + err.message)
      })
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.page = 1
      this.pageQuery()
    },
    handleCurrentChange(val) {
      this.page = val
      this.pageQuery()
    },
    handleStatus(empId, status){
      const params = {
        id: empId,
        status: !status ? 1 : 0
      }
      console.log(params)
      enableOrDisableEmployee(params).then(res =>{
        if (res.data.code === 1){
          this.$message.success('员工的账号状态修改成功！')
          this.pageQuery()
        }
      })
    },
    editEmployee(empId) {
      // if (empId === 1){
      //   this.$message.error('admin为系统管理员账号，不能修改！')
      //   return
      // }
      this.$router.push({
        path: '/employee/add',
        query: {id: empId}
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.disabled-text {
  color: #bac0cd !important;
}
</style>
