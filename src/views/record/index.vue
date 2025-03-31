<template>
  <div>
    <div class="header">
      <div v-show="getUser().role == '0'">
        <el-date-picker value-format="YYYY-MM-DD" v-model="queryParams.checkOutTime" type="date" placeholder="请选择" />
      </div>

      <el-button style="margin-left: 20px;" v-show="getUser().role == '0'" type="primary"
        @click="getRecordList">查询</el-button>
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

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">
      <video ref="video" autoplay style="width: 100%;height: 100%; object-fit: cover;"></video>

      <canvas ref="canvas" style="display: none;"></canvas>
      <div style="height:50px;display: flex; align-items: center; justify-content: center;">
        <el-button @click="startCamera">打开相机</el-button>
        <el-button @click="takePhoto">拍照</el-button>
        <el-button @click="stopCamera">关闭相机</el-button>
      </div>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible1 = false">取消</el-button>
          <el-button type="primary" @click="saveOrUpdate" :disabled="noClick">
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

const dialogVisible1 = ref(false)
const noClick = ref(true)
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

  axios.get('user/list', { params: { pageNum: 1, pageSize: 10000, role: '1' } }).then(res => {
    console.log(res)
    state.total = res.total
    state.left = state.total - state.data.total


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
  dialogVisible1.value = true

}

const saveOrUpdate = () => {
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


const video = ref(null);
const canvas = ref(null);
let stream = null;

// 打开摄像头
const startCamera = async () => {
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        width: { min: 640, ideal: 1280, max: 1920 },
        height: { min: 480, ideal: 720, max: 1080 },
        aspectRatio: 16 / 9, // 设置宽高比
      }, audio: false,
    });
    if (stream && stream.getVideoTracks().length > 0) {
      console.log("视频流已捕获");
      video.value.srcObject = stream;
    } else {
      console.error("视频流捕获失败");
    }

  } catch (error) {
    console.error("无法访问摄像头", error);
  }
};

// 拍照并转换为 File 对象
const takePhoto = () => {
  if (!canvas.value || !video.value) return;

  // 设置 canvas 大小为视频流的大小
  canvas.value.width = video.value.videoWidth;
  canvas.value.height = video.value.videoHeight;

  // 在 canvas 上绘制当前帧
  const context = canvas.value.getContext("2d");
  context.drawImage(video.value, 0, 0, canvas.value.width, canvas.value.height);

  // 将 canvas 转为 Blob
  canvas.value.toBlob((blob) => {
    if (blob) {
      blobToBase64(blob).then(res => {
        axios.post('user/faceCheck', { image: res }).then(res2 => {

          if(res2 == getUser().id){
            noClick.value=false
          }

        })
      })


    }
  }, "image/jpeg");
};


function blobToBase64(blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
}

const clearData = () => {
  stopCamera()
}
// 停止摄像头
const stopCamera = () => {
  if (stream) {
    stream.getTracks().forEach((track) => track.stop());
    stream = null;
  }
};

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