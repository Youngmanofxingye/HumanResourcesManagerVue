<template>
  <div class="manage">
    <el-dialog title="编辑" :visible.sync="editForm.isShow">
      <el-form label-width="100px" :model="editForm.formData">
        <el-form-item label="备注" label-width="140px" style="width:450px" prop="remark">
          <el-input type="textarea" placeholder="请输入" v-model.trim="editForm.formData.remark"
            :autosize="{ minRows: 2, maxRows: 4 }" maxlength="100" show-word-limit />
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="editForm.isShow = false"> 取消</el-button>
        <el-button type="primary" @click="confirmEdit"> 确定</el-button>
      </div>
    </el-dialog>

    <div style="margin-bottom: 10px">
      <el-upload :action="importApi" :headers="headers" accept="xlsx" :show-file-list="false" :multiple="false"
        :on-success="handleImportSuccess" style="display:inline-block;">
        <el-button type="success"> 导入
        </el-button>
      </el-upload>
      <el-button type="warning" @click="exportData" style="margin-left: 10px"> 导出
      </el-button>
      <el-upload :action="uploadApi" :headers="headers" :multiple="false" :show-file-list="false"
        :on-success="handleUploadSuccess" :limit="1" style="display:inline-block;margin-left: 10px">
        <el-button type="primary"> 上传
        </el-button>
      </el-upload>
      <el-popconfirm style="margin-left: 10px" confirm-button-text='确定' cancel-button-text='我再想想' title="你确定删除吗？"
        @confirm="handleDeleteBatch">
        <el-button type="danger" slot="reference"> 批量删除
        </el-button>
      </el-popconfirm>
      <el-button type="primary" @click="search" style="margin-left: 10px"> 搜索</el-button>
      <el-button type="danger" @click="reset"> 重置</el-button>

    </div>

    <!------------- 搜索 ---------------------->
    <div class="manage-header">
      <el-form label-width="auto" :model="searchForm.formData" :inline="true">
        <el-form-item label="原名称" prop="oldName">
          <el-input placeholder="请输入文件原名称" v-model.trim="searchForm.formData.oldName" prefix-icon="el-icon-search" />
        </el-form-item>
        <el-form-item label="上传者" prop="staffName">
          <el-input placeholder="请输入上传者" v-model.trim="searchForm.formData.staffName" prefix-icon="el-icon-search" />
        </el-form-item>
      </el-form>
    </div>
    <!------------------ 数据表格 ---------------->
    <div class="common-table">
      <el-table :data="table.tableData" height="85%" border stripe row-key="id" fit :header-cell-style="{
      background: '#eef1f6', color: '#606266',
      textAlign: 'center', fontWeight: 'bold', borderWidth: '3px'
    }" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="50" align="center" />
        <el-table-column prop="name" label="文件名" min-width="180" align="center" />
        <el-table-column prop="type" label="类型" min-width="125" align="center" />
        <el-table-column prop="oldName" label="文件原名称" min-width="150" align="center" />
        <el-table-column prop="size" label="文件大小（KB）" min-width="125" align="center" />
        <el-table-column prop="staffName" label="上传者" min-width="125" align="center" />
        <el-table-column prop="createTime" label="上传时间" min-width="150" align="center" />
        <el-table-column prop="remark" label="备注" min-width="200" align="center" />
        <el-table-column label="操作" width="360" fixed="right" align="center">
          <template slot-scope="scope">
            <el-button type="primary" @click="handleEdit(scope.row)"> 编辑</el-button>
            <el-popconfirm style="margin-left:10px;margin-right:10px" confirm-button-text='确定' cancel-button-text='我再想想'
              title="你确定删除吗？" @confirm="handleDelete(scope.row.id)">
              <el-button type="danger" slot="reference"> 删除</el-button>
            </el-popconfirm>
            <el-button type="warning" @click="download(scope.row.name)"> 下载</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination class="pager" layout="total,sizes,prev,pager,next,jumper"
        :page-size="table.pageConfig.size ? table.pageConfig.size : 10" :page-sizes="[5, 10, 15, 20]"
        :total="table.pageConfig.total" :current-page.sync="table.pageConfig.current" @size-change="handleSizeChange"
        @current-change="handleCurrentChange"></el-pagination>
    </div>
  </div>
</template>
<script>
import {
  deleteBatch,
  deleteOne,
  edit,
  getDownloadApi,
  getExportApi,
  getImportApi,
  getList,
  getUploadApi
} from '../../../api/docs'
import { mapState } from 'vuex'

export default {
  name: 'Docs',
  data() {
    return {
      editForm: {
        isShow: false,
        formData: {}
      },
      searchForm: {
        formData: {}
      },
      table: {
        tableData: [],
        pageConfig: {
          total: 10, // 记录总数
          current: 1, // 起始页
          size: 10 // 每页展示的记录数
        }
      },
      ids: []
    }
  },
  computed: {
    ...mapState('token', ['token']),
    headers() {
      return { token: this.token }
    },
    // 获取导入数据的接口
    importApi() {
      return getImportApi()
    },
    uploadApi() {
      return getUploadApi()
    }
  },
  methods: {
    handleDelete(id) {
      deleteOne(id).then(
        response => {
          if (response.code === 200) {
            this.$message.success('删除成功！')
            this.loading()
          } else {
            this.$message.error('删除失败！')
          }
        }
      )
    },
    handleDeleteBatch() {
      deleteBatch(this.ids).then(response => {
        if (response.code === 200) {
          this.$message.success('批量删除成功！')
          this.loading()
        } else {
          this.$message.error('批量删除失败！')
        }
      })
    },
    handleEdit(row) {
      this.editForm.isShow = true
      this.editForm.formData = row
    },
    confirmEdit() {
      edit(this.editForm.formData).then((response) => {
        if (response.code === 200) {
          this.$message.success('修改成功！')
          this.editForm.isShow = false
          this.loading()
        } else {
          this.$message.error('修改失败！')
        }
      })
    },
    search() {
      this.loading()
    },
    // 重置搜索表单
    reset() {
      this.searchForm.formData = {}
      this.loading()
    },
    handleSizeChange(size) {
      this.table.pageConfig.size = size
      this.loading()
    },
    handleCurrentChange(current) {
      this.table.pageConfig.current = current
      this.loading()
    },
    handleSelectionChange(list) {
      this.ids = list.map(item => item.id)
    },
    // 将数据渲染到模板
    loading() {
      getList({
        current: this.table.pageConfig.current,
        size: this.table.pageConfig.size,
        oldName: this.searchForm.formData.oldName,
        staffName: this.searchForm.formData.staffName
      }).then(response => {
        if (response.code === 200) {
          this.table.tableData = response.data.list
          this.table.pageConfig.total = response.data.total
        } else {
          this.$message.error(response.message)
        }
      })
    },
    // 导出数据
    exportData() {
      window.open(getExportApi())
    },
    handleImportSuccess(response) {
      if (response.code === 200) {
        this.$message.success('数据导入成功！')
        this.loading()
      } else {
        this.$message.error('数据导入失败！')
      }
    },
    handleUploadSuccess(response) {
      if (response.code === 200) {
        this.loading()
        this.$message.success('文件上传成功！')
      } else {
        this.$message.error('文件上传失败！')
      }
    },
    download(name) {
      window.open(getDownloadApi() + name)
    }
  },
  created() {
    this.loading()
  }
}
</script>
