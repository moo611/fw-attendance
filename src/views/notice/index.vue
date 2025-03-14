<template>
  <div>
    <div class="header">
      <el-button v-show="false" type="primary" @click="getNoticeList">查询</el-button>
      <el-button v-show="getUser().role == '0'" @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
     
      <el-table-column prop="title" label="标题" />
      <el-table-column prop="content" label="详情" />
      <el-table-column prop="createTime" label="创建时间" />


      <el-table-column label="操作" v-if="getUser().role == '0'">
        <template #default="scope">
          <el-button type="primary" @click="handleEdit(scope.$index, scope.row)">
            编辑
          </el-button>
          <el-button type="danger" @click="handleDel(scope.$index, scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="标题">
          <el-input v-model="form.title" />
        </el-form-item>
        <el-form-item label="内容">
          <el-input v-model="form.content"  />
        </el-form-item>
       
      
      </el-form>

      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible1 = false">取消</el-button>
          <el-button type="primary" @click="saveOrUpdate">
            确定
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="js" setup>
import { Plus } from '@element-plus/icons-vue'
import { reactive, ref } from 'vue';
import axios from '../../axios';
import { ElMessage } from 'element-plus';
import { getUser } from '../../utils/auth';
let mode = '0'
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,

})

const state = reactive({
  data: {},
  curNotice: {}
})

const form = reactive({
  title: '',
  content: '',
  id: '',
 
})
const dialogVisible1 = ref(false)

const clearData = () => {
  form.title = ''
  form.content = ''
  form.id = ''
 
  mode = '0'
}



const getNoticeList = () => {

  axios.get('notice/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}



const saveOrUpdate = () => {
  if (mode == '0') {


    axios.post('notice', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getNoticeList()

    })
  } else {
    axios.put('notice', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getNoticeList()

    })
  }


}

const copyValue = (src, target) => {
  // 遍历 target 中的 key，并将 src 对应属性赋值给 target
  Object.keys(target).forEach((key) => {
    if (src[key] !== undefined) {
      target[key] = src[key] // 仅赋值存在于 src 中的属性
    }
  })
}

const handleAdd = () => {
  mode = '0'
  dialogVisible1.value = true
}

const handleEdit = (index, row) => {
  mode = '1'
  copyValue(row, form)
  dialogVisible1.value = true
}
const handleDel = (index, row) => {
  axios.delete('notice/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getNoticeList()
  })
}


const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getNoticeList()
}

getNoticeList()

</script>

<style lang="css" scoped>
.header {
  height: 50px;
  position: relative;
  display: flex;
  align-items: center;
  /* margin-bottom: 30px; */

}

.btn-add {
  position: absolute;
  right: 20px;
}

.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}
.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
</style>