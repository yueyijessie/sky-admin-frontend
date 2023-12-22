<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right: 5px;">套餐名称：</label>
        <el-input v-model="name" placeholder="请输入套餐名称" style="width: 15%;" />
        <label style="margin-right: 10px; margin-left: 20px">套餐分类：</label>
        <el-select v-model="categoryId"
                   style="width: 14%"
                   placeholder="请选择"
                   clearable>
          <el-option v-for="item in mealCategoryList"
                     :key="item.id"
                     :label="item.name"
                     :value="item.id" />
        </el-select>
        <label style="margin-right: 10px; margin-left: 20px">售卖状态：</label>
        <el-select v-model="saleStatusVariable"
                   style="width: 14%"
                   placeholder="请选择"
                   clearable>
          <el-option v-for="item in saleStatus"
                     :key="item.value"
                     :label="item.label"
                     :value="item.value" />
        </el-select>
        <el-button class="normal-btn continue" @click="pageQuery">
          查询
        </el-button>

        <div class="tableLab">
          <span class="delBut non"
          @click="deleteHandle('批量', null)">批量删除</span>
          <el-button type="primary"
                     style="margin-left: 15px"
                     @click="() => this.$router.push('/setmeal/add')">
            + 新建套餐
          </el-button>
        </div>
      </div>


      <el-table v-if="records.length"
                :data="records"
                stripe
                class="tableBox"
                @selection-change="handleSelectionChange">
        <el-table-column type="selection"
                         width="25" />
        <el-table-column prop="name"
                         label="套餐名称" />
        <el-table-column prop="image"
                         label="图片">
          <template slot-scope="{ row }">
            <el-image style="width: 80px; height: 40px; border: none; cursor: pointer"
                      :src="row.image">
              <div slot="error"
                   class="image-slot">
                <img src="./../../assets/noImg.png"
                     style="width: auto; height: 40px; border: none">
              </div>
            </el-image>
          </template>
        </el-table-column>
        <el-table-column prop="categoryName"
                         label="套餐分类" />
        <el-table-column label="售价">
          <template slot-scope="scope">
            <span style="margin-right: 10px">￥{{ (scope.row.price ).toFixed(2)*100/100 }}</span>
          </template>
        </el-table-column>
        <el-table-column label="售卖状态">
          <template slot-scope="scope">
            <div class="tableColumn-status"
                 :class="{ 'stop-use': String(scope.row.status) === '0' }">
              {{ String(scope.row.status) === '0' ? '停售' : '启售' }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime"
                         label="最后操作时间" />
        <el-table-column label="操作"
                         width="250"
                         align="center">
          <template slot-scope="scope">
            <el-button type="text"
                       size="small"
                       class="blueBug"
                       @click="editMeal(scope.row.id)">
              修改
            </el-button>
            <el-button type="text"
                       size="small"
                       class="delBut"
                       @click="deleteHandle('单删', scope.row.id)">
              删除
            </el-button>
            <el-button type="text"
                       size="small"
                       class="non"
                       :class="{
                         blueBug: scope.row.status == '0',
                         delBut: scope.row.status != '0'
                       }"
                       @click="statusHandle(scope.row)">
              {{ scope.row.status == '0' ? '启售' : '停售' }}
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
import { getSetmealPage, enableOrDisableSetmeal, deleteSetmeal} from '@/api/setMeal'
import { getCategoryByType } from '@/api/category'
export default {
  data() {
    return {
      records: [], // 当前要展示的数据集合
      total: 0,
      page: 1,
      pageSize: 5,
      name: '',
      loading: true,
      mealStatus: '',
      saleStatusVariable:'',
      saleStatus: [ { value: 0, label: '停售'},
                    { value: 1, label: '启售'} ],
      mealCategoryList:[],
      categoryId: '',
      checkList: []
    }
  },
  created(){
    this.init()
  },
  methods:{
    init(){
      this.getCategory()
      this.pageQuery()
    },
    getCategory(){
      getCategoryByType({type: 2}).then(res => {
        if (res.data.code === 1) {
          this.mealCategoryList = res.data.data
          console.log(this.mealCategoryList)
        }
      })
    },
    // 分页查询
    pageQuery(){
      // 准备请求参数
      const params = {
        name: this.name,
        page: this.page,
        pageSize: this.pageSize,
        status: this.saleStatusVariable,
        categoryId: this.categoryId
      }
      // 发送ajax请求，访问后端服务，获取分页数据
      getSetmealPage(params).then(res => {
        if (res.data.code === 1){
          console.log(res.data.data)
          this.total = res.data.data.total
          this.records = res.data.data.records
          this.loading = false
        }
      }).catch(err => {
        this.$message.error('请求出错了：' + err.message)
      })
    },
    editMeal(st: string){
      this.$router.push({ path: '/setmeal/add', query: { id: st } })
    },
    deleteHandle(type: string, id: any) {
      if (type === '批量' && id === null) {
        if (this.checkList.length === 0) {
          return this.$message.error('请选择删除对象')
        }
      }
      this.$confirm('确认删除该菜品, 是否继续?', '确定删除', {
        confirmButtonText: '删除',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteSetmeal(type === '批量' ? this.checkList.join(',') : id)
          .then(res => {
            if (res && res.data && res.data.code === 1) {
              this.$message.success('删除成功！')
              this.init()
            } else {
              this.$message.error(res.data.msg)
            }
          })
          .catch(err => {
            this.$message.error('请求出错了：' + err.message)
          })
      })
    },
    handleSelectionChange(val: any) {
      let checkArr: any[] = []
      val.forEach((n: any) => {
        checkArr.push(n.id)
      })
      this.checkList = checkArr
    },
    statusHandle(row: any) {
      let params: any = {}
      if (typeof row === 'string') {
        if (this.checkList.length === 0) {
          this.$message.error('批量操作，请先勾选操作套餐！')
          return false
        }
        params.id = this.checkList.join(',')
        params.status = row
      } else {
        params.id = row.id
        params.status = row.status ? '0' : '1'
      }
      this.mealStatus = params
      this.$confirm('确认更改该套餐状态?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 起售停售---批量起售停售接口
        enableOrDisableSetmeal(this.mealStatus)
          .then(res => {
            if (res && res.data && res.data.code === 1) {
              this.$message.success('套餐状态已经更改成功！')
              this.init()
            } else {
              this.$message.error(res.data.msg)
            }
          })
          .catch(err => {
            this.$message.error('请求出错了：' + err.message)
          })
      })
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.page = 1
      this.init()
    },
    handleCurrentChange(val) {
      this.page = val
      this.init()
    }




  }
  
}
</script>
<style lang="scss">
.el-table-column--selection .cell {
  padding-left: 10px;
}
</style>
<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;

    .container {
      background: #fff;
      position: relative;
      z-index: 1;
      padding: 30px 28px;
      border-radius: 4px;

      .tableBar {
        margin-bottom: 20px;
        .tableLab {
          float: right;
          span {
            cursor: pointer;
            display: inline-block;
            font-size: 14px;
            padding: 0 20px;
            color: $gray-2;
          }
        }
      }

      .tableBox {
        width: 100%;
        border: 1px solid $gray-5;
        border-bottom: 0;
      }

      .pageList {
        text-align: center;
        margin-top: 30px;
      }
      //查询黑色按钮样式
      .normal-btn {
        background: #333333;
        color: white;
        margin-left: 20px;
      }
    }
  }
}
</style>
