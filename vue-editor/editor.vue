<template>
  <div class="hello">
    <vue-editor 
    v-model="content" 
    useCustomImageHandler
    @imageAdded='imageUpload'></vue-editor>
  </div>
</template>

<script>

import axios from 'axios'
import { VueEditor, Quill } from 'vue2-editor'

// import { ImageDrop } from 'quill-image-drop-module'
// import ImageResize from 'quill-image-resize-module'

// Quill.register('modules/imageResize', ImageResize);

export default {
  components:{
    VueEditor
  },
  props:{
    value:String
  },
  data () {
    return {
      actionUrl: 'https://upload.qiniup.com',
      content: ``,
      
    }
  },
  methods:{
    newGuid() {
      var guid = "";
      for (var i = 1; i <= 32; i++) {
        var n = Math.floor(Math.random() * 16.0).toString(16);
        guid += n;
        if ((i == 8) || (i == 12) || (i == 16) || (i == 20))
          guid += "-";
      }
      return guid;
    },
    // 获取七牛上传令牌
    getQNToken(){
      // 接口请求 token
      return axios.post('/api/Common/GetUploadToken').then(res=>{
        // console.log('token:',res)
        if(!res.Token) return this.$message.error('上传失败，请稍候再试')
        this.upToken = res.Token
        this.Domain = res.Domain
      }).catch(e=>{
        console.log(e)
        this.$message.error(e.message||'上传失败，请稍候再试')
      })
    },
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
    },
    // editor 上传图片动作
    imageUpload(file, Editor, cursorLocation, resetUploader){
      let _file = file
      const isPIC = _file.type === 'image/jpeg' ||　_file.type === 'image/png' ||　_file.type === 'image/gif'
      // console.log(_file.type)
      const isLt2M = _file.size / 1024 / 1024 < 2
      if( !isPIC ) return this.$message.error("请上传 jpg、jpeg、png、gif类型图片");
      if (!isLt2M) {
        this.$message.error("请上传2M以下的图片文件");
        return false;
      }

      this.getQNToken().then(result=>{
        return api.uploadToQN( this.actionUrl, this.upToken, _file)
      }).then(res=>{
        // console.log(res)
        let imgUrl = this.Domain + res.key
        // console.log('imgUrl:',imgUrl)

        Editor.insertEmbed(cursorLocation, 'image', imgUrl);
        resetUploader();
      }).catch(e=>{
        console.log(e)
      })

    },
  },
  watch:{
    value(){
      this.content = this.value
      console.log(this.value)
    },
    content(){
      this.$emit('change',{data: this.content})
    }
  }
}
</script>

