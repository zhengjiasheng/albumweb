<template>
  <div id="app">
    <!--相册组件-->
    <div id="albums" v-if="albumsStatus">
      <!--为子组件绑定事件@click.native="findPictures(item.id)"需要加一个native-->
      <Album @receiveAlbum="getAlbum"/>
    </div>
    <!--图片组件-->
    <div id="pictures" v-if="picturesStatus">
      <Pictures :alb_id="alb_id" @receivePictures="getPictures" @receivePicture2="getPictures2"/>
    </div>
    <!--画布组件-->
    <div id="canva" v-if="canvaStatus">
      <Canva :pictures="pictures" :picIndex="picIndex" @receiveCanva="getCanva"/>
    </div>
  </div>
</template>

<script type="javascript">
  import Album from './components/Album'
  import Pictures from "./components/Pictures"
  import Canva from './components/Canva'
  // import axios from 'axios'

  export default {
    name: 'App',
    data(){
      return{
        alb_id:0,            //被点击的相册的id：由Album子组件传递过来
        albumsStatus:true,   //相册页面状态
        picturesStatus:false,//图片页面状态
        canvaStatus:false,   //画布页面状态
        pictures:[],         //一个相册的全部图片,由Pictures子组件传递过来
        picIndex:0,          //被点击的图片在pictures数组中的下标，由Pictures子组件传递过来
      }
    },

    methods:{
      //接收Album子组件传过来的数据，并将之保存（alb_id:被点击的相册id  albumStatus:相册页面状态  picturesStatus:图片页面状态）
      getAlbum:function (alb_id, albumStatus, picturesStatus) {
        this.alb_id = alb_id;
        this.albumsStatus = albumStatus;
        this.picturesStatus = picturesStatus;
      },

      //接收Pictures子组件传过来的数据，并保存（albumStatus:相册页面状态  picturesStatus:图片页面状态）
      getPictures:function (albumStatus, picturesStatus) {
        this.albumsStatus = albumStatus;
        this.picturesStatus = picturesStatus;
      },

      //接收Pictures子组件传过来的数据，并保存
      getPictures2:function (index,pictures,picturesStatus,canvaStatus) {
        this.picIndex = index;
        this.pictures = pictures;
        this.picturesStatus = picturesStatus;
        this.canvaStatus = canvaStatus;
      },

      //接收Canva子组件传递过来的数据，并保存
      getCanva:function (picturesStatus,canvaStatus) {
        this.picturesStatus=picturesStatus;
        this.canvaStatus=canvaStatus;
      },
    },

    components: {
      Album,
      Pictures,
      Canva,
    }
  }
</script>

<style>
  #app {
    width: 100%;
    height: 100%;
  }

</style>
