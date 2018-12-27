<template>
  <div class="temp">
    <div class="pic-upload">
      <upload 
        @uploaded='onSuccess' 
        @remove="onRemove"
        :self='1'
        :src='IconUrl'/>
      <p class="sub-tip">小图（1:1）</p>
    </div>
    <div class="pic-upload">
      <upload 
        @uploaded='onSuccess' 
        @remove="onRemove"
        :self='2'
        :src='ImageUrl'/>
      <p class="sub-tip">大图（2:1）</p>
    </div>
  </div>  
</template>

<script>
import upload from './upload'
import Api from "@/sdk/sdk"

let api = new Api().health;

export default {
  components:{
    upload
  },
  data() {
    return {
      // 预设的传入url
      IconUrl:'https://healthcdn.51wnl-cq.com/3b43d3d8-14d3-2824-ad0a-56fb8e300d01.jpg',
      ImageUrl:'https://healthcdn.51wnl-cq.com/53ea85d8-e2cc-3143-2f00-93489b7ca913.jpg',
    };
  },
  methods:{
    onSuccess(data){
      console.log('from page success:',data)
      let { self,url } = data
      if(self == 1){
        this.form.FirstImageUrl = url
      }else if(self == 2) {
        this.form.SecondImageUrl = url
      }
    },
    onRemove(data){
      console.log('from page remove:',data)
      let { self } = data
      if(self == 1){
        this.form.FirstImageUrl = ''
      }else if(self == 2) {
        this.form.SecondImageUrl = ''
      }
    },
  },
};
</script>

<style scoped lang='less'>
  @import '../../assets/css/mixin.less';
  
  .filter{
    padding: 10px;
    .line{
      padding: 4px 0;
      .btn-group{
        float: right;
        margin-right: 20px;
      }
    }
    .bar{
      float: left;
      margin-right: 20px;
      &.fr{
        float: right ;
      }
    }
    .bar-title{
      color: @theme;
    }
  }
  .table{
    width: 100%;
    text-align: center;
  }

  .pic-upload{
    float: left;
    padding-bottom: 10px;
    margin-right: 20px;
    .sub-tip{
      text-align: center;
      line-height: 2;
    }
  }
  .table-pic{
    max-width: 90px;
    max-height: 90px;
  }
</style>
