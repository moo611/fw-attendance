<template>
  <div>
    <div class="header">
      <div v-show="getUser().role == '0'">
        <el-date-picker  value-format="YYYY-MM-DD" v-model="queryParams.checkOutTime"
        type="date" placeholder="请选择" />
      </div>
      
      <el-button style="margin-left: 20px;" v-show="getUser().role == '0'" type="primary" @click="getRecordList">查询</el-button>
      <el-button v-show="getUser().role == '1'" @click="handleAdd" type="primary" class="btn-add">打卡</el-button>

      <div class="right" v-show="getUser().role == '0'">
        <span style="margin: 10px;font-size: 14px;">打卡人数：{{ state.data.total }}</span>
        <span style="margin: 10px;font-size: 14px;">未打卡人数：{{ state.left }}</span>
        <span style="margin: 10px;font-size: 14px;">总人数：{{ state.total }}</span>
      </div>

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


const getToday = () => {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0'); // 月份从 0 开始，需要 +1
  const day = String(today.getDate()).padStart(2, '0');

  const formattedDate = `${year}-${month}-${day}`;
  console.log(formattedDate); // 例如：2023-10-05
  return formattedDate
}

const getTotal = () => {

  axios.get('user/list', { params: {pageNum:1,pageSize:10000,role:'1'} }).then(res => {
    console.log(res)
    state.total = res.total
    state.left = state.total-state.data.total
    

  })

}

const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  createBy: '',
  checkOutTime: ''
})
if (getUser().role == '0') {
  queryParams.checkOutTime = getToday()
}

const state = reactive({
  data: {},
  curRecord: {},
  total: 0,
  left: 0
})



const getRecordList = () => {
  if (getUser().role == '0') {
    queryParams.createBy = ''

  } else {
    queryParams.createBy = getUser().username
  }
  axios.post('record/list', queryParams).then(res => {
    
    state.data = res

    console.log(state.data)
    getTotal()

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

.right {
  position: absolute;
  right: 0;
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