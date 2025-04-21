<template>
  <div>
    <div class="header">
      <el-select v-model="queryParams.status" placeholder="请选择" style="width: 180px;margin-right: 10px;" @change="getApplyList">
        <el-option v-for="item in statusOptions" :key="item.value" :label="item.label" :value="item.value" ></el-option>
      </el-select>
     
      <el-button v-show="getUser().role == '1'" @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
      <el-table-column prop="nickname" label="申请人" />
      <el-table-column prop="reason" label="原因" />
      <el-table-column prop="startTime" label="开始日期" />
      <el-table-column prop="endTime" label="截止日期" />


      <el-table-column label="操作" v-if="queryParams.status == '0'">
        <template #default="scope">
          
          <el-button type="primary" @click="handleStatus('1', scope.row)" v-if="getUser().role == '0'">
            通过
          </el-button>
          <el-button type="danger" @click="handleStatus('2', scope.row)" v-if="getUser().role == '0'">
            拒绝
          </el-button>
          <el-button type="danger" @click="handleDel(scope.$index, scope.row)" v-if="getUser().role == '1'">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="原因">
          <el-input v-model="form.reason" />
        </el-form-item>
        <el-form-item label="起止日期">
          <el-date-picker v-model="dateRange" type="daterange" range-separator="至" start-placeholder="开始日期"
            end-placeholder="截止日期" @change="onDateChange" value-format="YYYY-MM-DD" />
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
  status: '0',
  username: '',
})

const statusOptions = [{label:'待审核',value:'0'},{label:'已审核',value:'1'},{label:'已拒绝',value:'2'}]

const state = reactive({
  data: {},
  curApply: {}
})



const form = reactive({
  reason: '',
  startTime: '',
  endTime:'',
  id: '',

})
const dateRange = ref([])
const dialogVisible1 = ref(false)

const onDateChange = (value) => {
  console.log(value)
  form.startTime = value[0]
  form.endTime = value[1]
  console.log(form)
}

const clearData = () => {
  form.reason = ''
  form.startTime = ''
  form.endTime = ''
  form.id = ''
  dateRange.value = []
  mode = '0'
}



const getApplyList = () => {
  if (getUser().role == '0') {
    queryParams.username = ''
  } else {
    queryParams.username = getUser().username
  }
  axios.get('apply/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}


const saveOrUpdate = () => {
  form.username = getUser().username
  if (mode == '0') {

    axios.post('apply', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getApplyList()

    })
  } else {
    axios.put('apply', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getApplyList()

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

const handleStatus = (status, row) => {
  row.status = status
  axios.put('apply', row).then(res => {
    ElMessage.success('更新成功')
    getApplyList()
  })

}

const handleDel = (index, row) => {
  axios.delete('apply/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getApplyList()
  })
}


const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getApplyList()
}

getApplyList()

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