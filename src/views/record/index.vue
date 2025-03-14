<template>
  <div>
    <div class="header">
      <el-button v-show="false" type="primary" @click="getRecordList">查询</el-button>
      <el-button v-show="getUser().role == '1'" @click="handleAdd" type="primary" class="btn-add">打卡</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">

      <el-table-column prop="createName" label="打卡人" />
      <el-table-column prop="checkInTime" label="上岗时间" />
      <el-table-column prop="checkOutTime" label="离岗时间" />

    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />


  </div>
</template>

<script lang="js" setup>
import { Plus } from '@element-plus/icons-vue'
import { reactive, ref } from 'vue';
import axios from '../../axios';
import { ElMessage } from 'element-plus';
import { getUser } from '../../utils/auth';

const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  createBy:''
})

const state = reactive({
  data: {},
  curRecord: {}
})



const getRecordList = () => {
  if(getUser().role == '0'){
    queryParams.createBy = ''
  }else{
    queryParams.createBy = getUser().username
  }
  axios.get('record/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}



const handleAdd = () => {
  axios.post('record/registry').then(res => {

    
    ElMessage.success('打卡成功')
    getRecordList()

  })
}

const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getRecordList()
}

getRecordList()

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