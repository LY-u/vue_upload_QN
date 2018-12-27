<template>
  <div class="temp">
    <editor :value='content' @change='editorChange'></editor>
  </div>  
</template>

<script>
import editor from '@/components/editor'

export default {
  components:{
    editor
  },
  data() {
    return {
      content:'请输入。。。',
    };
  },
  methods:{
    editorChange(res){
      let data = res.data
      // console.log(data)
      this.checkImg()
      this.content = data
    },
  },
};
</script>

