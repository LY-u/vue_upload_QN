<template>
  <div :class="{hiddenPlus:!holder}">
    <el-upload
      action="action"
      accept='image/jpeg,image/gif,image/png'
      list-type="picture-card"
      :file-list='fileList'
      :limit='1'
      :on-remove="removePicture"
      :on-success="uploadSuccess"
      :before-upload="beforeUpload"
      :http-request='uploadQN'>
      <i class="el-icon-plus"></i>
    </el-upload>
  </div>
</template>

<script>

class Health extends SdkBase {
  constructor (key) {
    super(key)
    // this.router = '/api/hotel'
  }
  getUploadToken(){
    return this._post('/api/Common/GetUploadToken')
  }
  uploadToQN(url, token, file){
    let name = this.newGuid()
    let types = file.name.split('.')
    let file_type = types[types.length-1]
    name = name + '.' + file_type
    let formData = new FormData()
    formData.append('file', file)
    formData.append('key', name)
    formData.append("token", token)
    let config = {
      headers: { 'Content-Type': 'multipart/form-data' },
    }
    return axios.post(url, formData, config)
  }
}
  
let api = new Api().health;

export default {
  props:{
    src: String,          // 预设的传入URL
    self:null,            // 给组件自定义的数据，由父组件带入，在 emit 时返回给父组件
  },
  data() {
    return {
      fileList:[],          // 图片列表
      holder:true,          // 是否显示加号
      actionUrl:'https://upload.qiniup.com',      // upload url
    };
  },
  methods:{
    // 1. 获取 上传 token
    getQN(){
      return api.getUploadToken().then(res=>{
        // console.log('token:',res)
        if(!res.Token) return this.$message.error('上传失败，请稍候再试')
        this.upToken = res.Token        // token 
        this.Domain = res.Domain        // 返回图片的前半段url
      }).catch(e=>{
        console.log(e)
        this.$message.error(e.message||'上传失败，请稍候再试')
      })
    },
    // 2. 通过 http-request 修改上传请求
    uploadQN(param){
      // file 文件
      let _file = param.file
      // 验证文件类型
      const isPIC = _file.type === 'image/jpeg' ||　_file.type === 'image/png' ||　_file.type === 'image/gif'
      // console.log(_file.type)
      // 限制大小
      const isLt2M = _file.size / 1024 / 1024 < 2
      if( !isPIC ) return this.$message.error("请上传 jpg、jpeg、png、gif类型图片");
      if (!isLt2M) {
        this.$message.error("请上传2M以下的图片文件");
        return false;
      }

      // 请求 token
      this.getQN().then(result=>{
        // 上传到 七牛
        return api.uploadToQN( this.actionUrl, this.upToken, _file)
      }).then(res=>{
        // console.log(res)
        let imgUrl = this.Domain + res.key
        console.log(imgUrl)
        // 触发父组件事件
        this.$emit('uploaded',{
          url: imgUrl,
          self: this.self,
        })
        this.fileList = [{url: imgUrl}]
      }).catch(e=>{
        console.log(e)
      })
    },
    uploadSuccess(res, file, fileList){
      console.log('success',res)
    },
    beforeUpload(){
      this.holder = false
    },
    removePicture(file, fileList){
      // 删除图片，并向父组件传递
      this.holder = true
      this.$emit('remove',{
        self: this.self,
      })
    },
    setUrl(){
      // 将 props 传来的图片地址 分发到 filelist 里
      if(this.src) {
        this.fileList = [{url: this.src}]
        this.holder = false
      }else{
        this.fileList = []
        this.holder = true
      }
    },
  },
  created(){
    this.setUrl()
  },
  watch: {
    src() {
      this.setUrl()
    }
  }
};
</script>

<style lang='less'>
  // @import '../assets/css/mixin.less';
  .hiddenPlus .el-upload--picture-card{
    display: none;
  }
  .el-upload-list--picture-card .el-upload-list__item{
    margin: 0;
  }
  .el-upload-list--picture-card{
    display: inline-block;
    line-height: 0
  }
</style>
