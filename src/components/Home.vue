// components/Home.vue

<template>
  <div>
    <center class="title">请上传垃圾照片</center>
     <el-divider>从摄像头获取图片</el-divider>
    <center>
       <!--开启摄像头-->
       <div>
         <el-button size="mini" type="primary" @click="callCamera">打开摄像头</el-button>
       </div>
        <!--canvas截取流-->
        <canvas ref="canvas" width="240" height="240"></canvas>
        <!--图片展示-->
        <video ref="video" width="240" height="240" autoplay></video>
        <div>
          <!--确认-->
        <el-button size="mini" type="primary" @click="photograph">拍照</el-button>
        <!--关闭-->
        <el-button size="mini" type="danger" @click="closeCamera">关闭</el-button>
        </div>
        
    </center>
    <el-divider>或上传本地上传图片</el-divider>
    <el-upload
      v-loading="loading"
      element-loading-text="识别中"
      class="avatar-uploader"
      :action="action"
      list-type="picture-card"
      :limit='1'
      :on-success="handleAvatarSuccess"
      :on-error="handleAvatarError"
      :before-upload="beforeAvatarUpload"
      :auto-upload="false"
      ref="upload"
    >
      <img v-if="imageUrl" :src="imageUrl" class="avatar" />
      <!-- <i v-else class="el-icon-plus avatar-uploader-icon"></i> -->
    </el-upload>
    <center style="margin-top:20px">
      识别结果为：
      <el-input
        style="width:150px;margin-top:20px"
        placeholder=""
        v-model="result"
        :disabled="true"
      >
      </el-input>
      <el-button size="mini" type="primary" @click="submitUpload">识别</el-button>
    </center>
  </div>
</template>
<script>
export default {
  data() {
    return {
      imageUrl: "",
      action: "",
      result: "",
      loading: false,
      headImgSrc: '',
      img:''
    };
  },
  mounted() {
    this.action = this.GLOBAL.host + "/rubbish";// 后台访问地址，flask为http://127.0.0.1:5000/rubbish
  },
  methods: {
    submitUpload() {
      if(this.img!=''){
        this.loading = true;
        let param = new FormData()  // 创建form对象
      param.append('file', this.img, this.img.name)  // 通过append向form对象添加数据
      let config = {
        headers: {'Content-Type': 'multipart/form-data'}
      }
      this.$axios.post(this.action,param, config)
        .then(response => {
          this.result = response.data.data;
          this.loading = false;
          this.img=''
        })
      }else{
        this.$refs.upload.submit();
      }
    },
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
      this.result = res.data;
      this.loading = false;
    },
    handleAvatarError(res, file) {
      
      this.loading = false;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 10;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
        this.loading = false;
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 10MB!");
        this.loading = false;
      }
      this.loading = true;
      return isJPG && isLt2M;
    },
    dataURLtoFile(dataurl, filename) {
        var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
            bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
        while(n--){
            u8arr[n] = bstr.charCodeAt(n);
        }
        return new File([u8arr], filename, {type:mime});
    },
    // 调用摄像头
    callCamera () {
      // H5调用电脑摄像头API
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then(success => {
        // 摄像头开启成功
        this.$refs['video'].srcObject = success
        // 实时拍照效果
        this.$refs['video'].play()
      }).catch(error => {
        console.error('摄像头开启失败，请检查摄像头是否可用！')
      })
    },
    // 拍照
    photograph () {
      let ctx = this.$refs['canvas'].getContext('2d')
      // 把当前视频帧内容渲染到canvas上
      ctx.drawImage(this.$refs['video'], 0, 0, 240, 240)
      // 转base64格式、图片格式转换、图片质量压缩
      let imgBase64 = this.$refs['canvas'].toDataURL('image/jpeg', 0.7)
      this.img = this.dataURLtoFile(imgBase64,'img')
    },
    // 关闭摄像头
    closeCamera () {
      this.img=''
      let ctx = this.$refs['canvas'].getContext('2d')
      ctx.clearRect(0,0,240, 240);
      if (!this.$refs['video'].srcObject) {
        this.dialogCamera = false
        return
      }
      let stream = this.$refs['video'].srcObject
      let tracks = stream.getTracks()
      tracks.forEach(track => {
        track.stop()
      })
      this.$refs['video'].srcObject = null
    },
  }
};
</script>
<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
