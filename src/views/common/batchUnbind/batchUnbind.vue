<template>
  <div class="mod-user">
    <el-form :inline="true">
      <el-form-item>
        <el-upload
          action="111"
          :show-file-list="false"
          :http-request="doUpload">
          <el-button size="small" type="primary">上传Excle</el-button>
          <div slot="tip" class="el-upload__tip">只能上传xlsx/xls 文件</div>
        </el-upload>
      </el-form-item>
    </el-form>
    <!-- 添加,编辑弹窗 -->
    <el-dialog title="批量解绑" :visible.sync="addVisible" width="800px" :close-on-click-modal="false">
      <el-table
        :data="dataList"
        border
        height="480px"
        v-loading="dataListLoading"
        style="width: 100%;">
        <el-table-column type="index" label="序号" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column prop="no" header-align="center" align="center" label="产品序列号"></el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitAddForm('addForm')" :loading="addLoading">确 认</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  import { uploadExcle, unbindList } from '@/api/baseApi'
import { format } from 'path'
  export default {
    data () {
      return {
        dataList: [],
        dataListBackUp: [],
        pageNum: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addLoading: false,
        addVisible: false,
      }
    },
    mounted () {
  
    },
    methods: {
      // 上传excle
      doUpload (data) {
        const testmsg= data.file.name.substring(data.file.name.lastIndexOf('.') + 1)
        const extension = testmsg === 'xls'
        const extension2 = testmsg === 'xlsx'
        const isLt2M = data.file.size / 1024 / 1024 < 10
        if(!extension && !extension2) {
          this.$message({ message: '上传文件只能是 xls、xlsx格式!', type: 'warning' })
          return false
        }
        if(!isLt2M) {
          this.$message({message: '上传文件大小不能超过 10MB!', type: 'warning'})
          return false
        }

        let formdata = new FormData()
        formdata.append('file', data.file)
        this.$http.post(uploadExcle, formdata).then(res => {
          if (res.data.code === '200') {
            this.$notify({title: '提示', message: '文件上传成功', type: 'success'})
            const tableList = []
            const resData = res.data.data
            this.dataListBackUp = res.data.data
            resData.map(item => {
              tableList.push({ no: item })
            })
            this.dataList = tableList
            this.addVisible = true
          } else {
            this.$notify({title: '提示', message: res.data.msg, type: 'error'})
          }
        })
      },
      // 添加修改提交
      submitAddForm (formName) {
        this.$confirm(`是否确认提交?`, '提示', {type: 'warning'}).then(() => {
          this.addLoading = true
          // 新增提交
          this.$http.post(unbindList, this.dataListBackUp).then(res => {
            if (res.data.code === '200') {
              this.$notify({title: '提示', message: `成功解绑${res.data.data}条！`, type: 'success'})
              this.addVisible = false
            } else {
              this.$notify({title: '提示', message: res.data.msg, type: 'error'})
            }
          }).catch(error => {
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
</style>
