<template>
  <div id="album">
    <!--添加相册-->
    <div id="add">
      <!--添加相册div的背景图片-->
      <img src="../assets/add.jpg" alt="" id="add_albums" @click="addAlbum">
    </div>
    <!--相册：通过循环遍历相册数组，把所有相册展示出来-->
    <div id="oneAlbum" v-for="(item,index) in albums">
      <!--相册名字-->
      <h1 style="z-index: 1;position: absolute;" align="center">{{ item.alb_name }}</h1>
      <!--相册的删除按钮-->
      <button style="z-index: 1;position: absolute;margin-left: 185px" @click="removeAlbum(item.alb_id)">删除</button>
      <!--相册的封面：封面为最新添加的图片-->
      <img v-bind:src="'http://192.168.130.153:8887/images/'+item.default_picture" id="onePicture" @click="turnPictures(item.alb_id)">
    </div>
  </div>
</template>

<script>
import axios from 'axios'

  export default {
    data () {
      return {
        albums:[],      //全部相册
      }
    },

    //生命周期方法：页面加载完成后执行
    mounted:function(){
      //调用查询全部相册的方法
      this.findAlbums();
    },

    methods:{
      //*查询全部相册
      findAlbums(){
        var that = this;
        axios.get("http://192.168.130.153:8887/AlbumsController/findAll")
          .then(function (response) {
            that.albums = response.data;  //查询出的数据（全部相册）
            // console.log(that.albums);
          },function (err) {
            console.log(err);
          });
      },

      //添加相册
      addAlbum:function () {
        var that = this;
        var name = prompt("请输入相册名");    //输入对话框
        if (name != null) {
          axios.get("http://192.168.130.153:8887/AlbumsController/addAlbum?name="+name)
            .then(function (response) {
              //添加相册之后在调用查询全部相册的方法，更新页面
              that.findAlbums();
            },function (err) {
              console.log(err);
            })
        }
      },

      //删除相册（参数：要删除的相册的id）
      removeAlbum:function (id) {
        var that = this;
        axios.get("http://192.168.130.153:8887/AlbumsController/deleteAlbum?id="+id)
          .then(function (response) {
            // console.log(id);
            //删除相册之后在调用查询全部相册的方法，更新页面
            that.findAlbums();
          },function (err) {
            console.log(err);
          });
      },

      /* 跳转到图片页面
       * 实质上是向主组件传参数，然后把相册页面隐藏，让图片页面展示
       * 同时要把相册的id传过去，用于图片页面查询该相册的全部图片 */
      turnPictures:function (alb_id) {
        /***Album向App传了三个参数，第一个是相册id，第二个是相册页面的状态（隐藏），第三个是图片页面的状态（展示）***/
        this.$emit("receiveAlbum",alb_id,false,true);
      }
    }

  }
</script>

<style>
  #add{
    float: left;
    height: 300px;
    width: 228px;
    border: 10px solid #C0C0C0;
    display:inline-block
  }
  #add_albums{
    width: 100%;
    height: 100%;
  }
  #oneAlbum{
    width: 228px;
    height: 300px;
    background-color: white;
    border: 10px solid #C0C0C0;
    display:inline-block

  }
  #onePicture{
    width: 100%;
    height: 100%;
  }
</style>
