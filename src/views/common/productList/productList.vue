<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="注册手机号">
        <el-input v-model="dataForm.mobile" placeholder="请输入注册手机号" clearable></el-input>
      </el-form-item>
      <el-form-item label="产品序列号">
        <el-input v-model="dataForm.serialNumber" placeholder="请输入产品序列号" clearable></el-input>
      </el-form-item>
      <el-form-item label="购买日期">
        <el-date-picker
          v-model="dataForm.purchaseDate"
          type="daterange"
          range-separator="to"
          start-placeholder="开始日期"
          end-placeholder="结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="创建日期">
        <el-date-picker
          v-model="dataForm.createDate"
          type="daterange"
          range-separator="to"
          start-placeholder="开始日期"
          end-placeholder="结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()" type="primary">查询</el-button>
        <!-- <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button> -->
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column type="index" label="序号" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="serialNumber" header-align="center" align="center" label="序列号"></el-table-column>
      <el-table-column prop="mobile" header-align="center" align="center" label="手机号"></el-table-column>
      <el-table-column prop="purchaseDate" header-align="center" align="center" label="购买日期"></el-table-column>
      <el-table-column prop="statusDesc" header-align="center" align="center" label="状态"></el-table-column>
      <el-table-column prop="statusDesc" header-align="center" align="center" label="查看小票">
        <template slot-scope="scope">
          <img v-if="scope.row.status==1 || scope.row.status==2 || scope.row.status==3 || scope.row.status==4" :src="scope.row.invoiceUrl" preview="0" class="img_wrap" alt="小票">
        </template>
      </el-table-column>
      <el-table-column prop="repairEndDate" header-align="center" align="center" label="保修到期日"></el-table-column>
      <el-table-column prop="repairExtendDate" header-align="center" align="center" label="延保到期日">
        <template slot-scope="scope">
          {{ scope.row.repairExtendDate ? scope.row.repairExtendDate : '未延保' }}
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="240"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.status==1" type="text" size="small" @click="addOrUpdateHandle(scope.row)">审核</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageNum"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination> -->
    <!-- 添加,编辑弹窗 -->
    <el-dialog :title="!addForm.id ? '新增' : '修改'" :visible.sync="addVisible" width="400px" :close-on-click-modal="false">
      <el-form :model="addForm" :rules="rules" ref="addForm" label-width="100px">
        <el-form-item label="状态" prop="type">
          <el-radio v-model="addForm.type" label="1">通过</el-radio>
          <el-radio v-model="addForm.type" label="0">不通过</el-radio>
        </el-form-item>
        <el-form-item label="购买时间" prop="purchaseDate">
          <el-date-picker
            v-model="addForm.purchaseDate"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="选择日期">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitAddForm('addForm')" :loading="addLoading">提 交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { queryProductList, verify } from '@/api/baseApi'
import { format } from 'path'
  export default {
    data () {
      return {
        dataForm: {
          mobile: '',
          serialNumber: '',
          purchaseDate: [],
          createDate: []
        },
        dataList: [],
        pageNum: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addForm: {
          id: 0,
          type: '1',
          purchaseDate: ''
        },
        addLoading: false,
        rules: {},
        addVisible: false,
      }
    },
    activated () {
      this.getDataList()
    },
    mounted () {
  
    },
    methods: {
      classChange () {
        this.getDataList()
      },
      // 获取数据列表
      getDataList () {
        let loginInfo = JSON.parse(sessionStorage.getItem('headerInfo'))
        console.log(loginInfo)
        this.dataListLoading = true
        let params = {
          mobile: this.dataForm.mobile,
          serialNumber: this.dataForm.serialNumber,
          startPurchaseDate: this.dataForm.purchaseDate.length ? this.dataForm.purchaseDate[0] : '',
          endPurchaseDate: this.dataForm.purchaseDate.length ? this.dataForm.purchaseDate[1] : '',
          startCreateDate: this.dataForm.createDate.length ? this.dataForm.createDate[0] : '',
          endCreateDate: this.dataForm.createDate.length ? this.dataForm.createDate[1] : ''
        }
        this.$http.post(queryProductList, params).then(res => {
          console.log(res.data)
          this.dataList = res.data.data
          this.totalPage = res.data.total
        }).catch(error => {
          console.log(error)
        }).finally(() => {
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageNum = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageNum = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增, 编辑
      addOrUpdateHandle (row) {
        console.log(row)
        // 参数为空或者undefined则为新增 否则为修改
        if (row == '' || row == undefined) {
          this.addForm.type = '1',
          this.addForm.purchaseDate = ''
          this.addForm.id = 0
        } else {
          this.addForm.id = row.id
          // this.addForm.status = row.status
        }
        this.addVisible = true
      },
      // 添加修改提交
      submitAddForm (formName) {
        this.$confirm(`确认审核?`, '提示', {type: 'warning'}).then(() => {
          this.addLoading = true
          let mType = ''
          if (this.addForm.type == '1') {
            if (this.addForm.purchaseDate) {
              mType = 3
            } else {
              mType = 2
            }
          } else {
            mType = 4
          }
           let loginInfo = JSON.parse(sessionStorage.getItem('loginInfo'))
          let params = {
            adminId: loginInfo.id,
            id: this.addForm.id,
            status: mType,
            purchaseDate: this.addForm.purchaseDate
          }
          // 新增提交
          this.$http.post(verify, params).then(res => {
            if (res.data.code == '200') {
              this.$notify({title: '提示', message: '审核成功！', type: 'success'})
              this.addVisible = false
              this.getDataList()
            } else {
              this.$notify({title: '提示', message: res.data.msg, type: 'error'})
            }
          }).catch(error => {
            this.$notify({title: '提示', message: '审核失败！', type: 'error'})
            console.log(error)
          }).finally(() => {
            this.addLoading = false
          })
        }).catch(() => {})
      }
    }
  }
</script>
<style lang="css" scoped>
.avatar-uploader {
  width: 148px;
  height: 148px;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  overflow: hidden;
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 148px;
  height: 148px;
  line-height: 148px;
  text-align: center;
}
.avatar {
  width: 148px;
  height: 148px;
  display: block;
}
.img_wrap {
  width: 40px;
  height: 40px;
  border-radius: 4px;
  overflow: hidden;
}
</style>
