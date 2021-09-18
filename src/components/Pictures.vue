<template>
  <div id="pic">
    <!--返回按钮:返回相册页面-->
    <button id="back" @click="back">返回</button>
    <!--图片列表:展示一个相册的所有图片-->
    <ul id="pictures">
      <!--上传图片:通过input标签向服务器上传图片-->
      <li id="addPicture">
        <div id="btn1">
          <span>选择文件</span>
          <!--input标签:用于上传图片,类型是file,限制上传文件的格式为图片(accept="image/*")-->
          <input accept="image/*" name="img" id="upload_file" type="file" class="btn1">
        </div>
        <!--提交按钮:提交选择的图片,实现图片上传-->
        <button class="btn2" @click="addPicture">提交</button>
      </li>
      <!--通过循环遍历pictures数组,展示一个相册的全部图片-->
      <li v-for="(item,index) in pictures">
        <!--删除按钮:用于删除图片-->
        <button id="deleteBtn" @click="deletePicture(item.pic_id,item.pic_name)">删除</button>
        <!--展示的图片-->
        <img :src="'http://192.168.130.153:8887/images/'+item.pic_name" id="album_img" @click="turnCanva(index)">
      </li>
    </ul>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name:"Pictures",
    props:{
      //被点击的相册的id
      alb_id:{
        type:Number
      }
    },

    data(){
      return{
        pictures:[],   //一个相册的全部图片  需要传给画布（Canvas）页面，先传给App页面，再由App页面传给画布页面
      }
    },

    //生命周期方法:组件创建时调用
    created:function(){
      //调用findPictures方法，查询该相册的全部图片，相册的id作为方法参数传递过去
      this.findPictures(this.alb_id);
    },

    methods:{
      //查询一个相册的全部图片
      findPictures:function (alb_id) {
        var that = this;
        axios.get("http://192.168.130.153:8887/PicturesController/findPictures?id="+alb_id)
          .then(function (response) {
            that.pictures = response.data;
            // console.log(that.pictures);
          },function (err) {
            console.log(err);
          })
      },

      //添加图片*****图片上传******
      addPicture:function () {
        var that = this;
        var aa = document.getElementById("upload_file");
        if (aa.value == ""){         //判断是否选择图片
          alert("请选择图片！");     //没选择
        }else {                      //选择了
          //获取选择的图片
          var file = document.getElementById("upload_file").files[0];
          // 创建form对象
          var formdata1 = new FormData();
          // 通过append方法向form对象添加数据,可以通过append继续添加数据
          formdata1.append('img', file, file.name);  //参数：img：key   file:value
          //或formdata1.append('img',file);
          //添加请求头
          let config = {
            headers: {'Content-Type': 'multipart/form-data'}
          };
          //发送post请求，进行图片上传(参数id为相册的id）
          axios.post('http://192.168.130.153:8887/PicturesController/addPicture?id='+that.alb_id, formdata1, config)
            .then(function (response) {
            //上传图片之后，再调用查询全部图片的方法，来更新页面
            that.findPictures(that.alb_id);
            // console.log(response.data);
          })//图片上传
        }
      },

      //删除图片(参数：1.图片id  2.图片名字）
      deletePicture:function (id,name) {
        var that = this;
        axios.get("http://192.168.130.153:8887/PicturesController/deletePicture?id="+id+"&&name="+name)
          .then(function (response) {
            //删除图片之后，再调用查询全部图片的方法，来更新页面
            that.findPictures(that.alb_id);
            // console.log(response.data);
          },function (err) {
            console.log(err);
          })
      },

      //返回相册
      back:function () {
        // this.findAlbums();
        //子组件Pictures向父组件App传两个参数：1.相册页面状态（显示true）  2.图片页面状态（隐藏false）
        this.$emit("receivePictures",true,false);
      },

      //跳转到画布页面
      //子组件Pictures向父组件App传了四个参数：
      /*1.点击的图片在pictures数组中的下标：用于画布页面图片的展示和图片的切换
      **2.pictures数组：保存该相册的全部图片
      **3.图片页面的状态：false(隐藏)
      * 4.画布页面的状态：true（显示）**/
      turnCanva:function (index) {
        this.$emit("receivePicture2",index,this.pictures,false,true);
      }
    }

  }
</script>

<style>
  #pictures{
    padding-left: 2px;
    width: 100%;
  }
  li{
    height: 290px;
    width: 228px;
    border: 10px solid #C0C0C0;
    display:inline-block
  }
  #album_img{
    width: 100%;
    height: 100%;
  }
  #addPicture{
    position: relative;
    float: left;
    height: 290px;
    width: 228px;
    border: 10px solid #C0C0C0;
    display:inline-block;
    background-color: aqua;
  }
  #back{
    position: sticky;
    top: 10px;
    left: 96%;
    padding: 5px;
    background-color: #cae8ca;
    border: 0 solid white;
  }
  #pic{
    margin-top: 0px;
  }
  .btn1{
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    right: 0;
    opacity: 0;
  }
  #btn1{
    position: relative;
    width: 228px;
    height: 50px;
    line-height: 50px;
    background-color: pink;
    color: #fff;
    text-align: center;
    top: 30px;
  }
  .btn2{
    position: absolute;
    top: 148px;
    left: 71px;
    width: 85px;
    height: 85px;
    background-color: #CCFFFF;
  }
  .btn2:hover{
    background-color: pink;
  }
  #deleteBtn{
    z-index: 1;
    position: absolute;
    margin-left: 185px
  }
</style>
