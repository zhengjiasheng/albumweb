<template>
  <div id="canvas">
    <!--返回按钮：返回图片页面-->
    <button @click="backPicture" id="backBtn">返回</button>
    <!--图片和画布模块-->
    <div  id="canva_picture">
      <!--图片向左切换-->
      <span id="left" @click="subIndex" v-if="pic_index>0">&lt;</span>
      <!--图片向右切换-->
      <span id="right" @click="addIndex" v-if="pic_index<pictures.length-1">&gt;</span>
      <!--图片:位于下层-->
      <img :src="'http://192.168.130.153:8887/images/'+pictures[pic_index].pic_name" alt="" id="one_picture" width="700px" height="400px"/>
      <!--画布：位于上层-->
      <canvas id="myCanvas" ref="myCanvas" width="700px" height="400px"></canvas>
    </div>
    <!--操作模块-->
    <div id="operation">
      <!--画笔按钮-->
      <button @click="paintBrush1()">画笔</button>
      <!--清空按钮-->
      <button @click="clearAll()">清空</button>
      <!--橡皮擦按钮-->
      <button @click="clearLine()">橡皮擦</button>
      <!--颜色选择器：用于改变画笔颜色-->
      <input type="color" v-model="paintBrushColor" id="chCloor">
      <!--用于显示或隐藏颜色选择器的按钮:设置其背景颜色和画笔颜色一致-->
      <input type="button" id="isColorSelector" @click="isColorSelector" :style="{'background':+' '+paintBrushColor}">
      <!--滑块：用于改变画笔粗细-->
      <el-slider v-model="paintBrushWidth" id="slider"></el-slider>
    </div>
      <!--颜色选择器：ColorSelector组件-->
      <ColorSelector v-show="csStatus" @receiveColor="getColor" @receiveStatus="getStatus"/>
  </div>
</template>

<script>
  import ColorSelector from './ColorSelector'
  import axios from 'axios'

  export default {
    name:'Canva',
    props:{
      //一个相册的全部图片，由父组件App传递过来
      pictures:{
        type:Array
      },
      //被点击的图片在pictures数组中的下标，也是画布页面首先展示的图片
      picIndex:{
        type: Number
      },
    },

    data(){
      return{
        paintBrushColor:"#000000",    //画笔颜色
        paintBrushWidth:2,            //画笔粗细
        lineArr:[],                   //存储线条的数组
        pointArr:[],                  //存储坐标点的数组
        pic_index:0,                  //被点击的图片在pictures数组中的下标，也是画布页面首先展示的图片，来自于picIndex
        csStatus:false,               //颜色选择器的状态
      }
    },

    components:{
      ColorSelector,                  //颜色选择器
    },

    //生命周期方法：页面加载完成后执行
    mounted:function(){
      this.canvas = this.$refs.myCanvas;      //获取画布元素
      this.ctx = this.canvas.getContext("2d");//把画布变成二维
      this.pic_index=this.picIndex;           //将picIndex保存到pic_index,用于图片切换（注：在子组件中不能修改父组件传过来的变量）
      this.readPoint();                       //调用查询笔记的方法
    },
    methods:{
      //显示或隐藏颜色选择器的方法
      isColorSelector:function(){
        this.csStatus = !this.csStatus;
      },

      //获取子组件ColorSelector传递过来的值
      // 参数：color：通过颜色选择器选择的颜色，赋值给paintBrushColor（画笔颜色）
      //       csStatus:颜色选择器状态（隐藏）
      getColor:function(color,csStatus){
        this.paintBrushColor=color;
        this.csStatus=csStatus;
      },

      //获取子组件ColorSelector传递过来的颜色选择器的状态（false） 关闭颜色选择器
      getStatus:function(csStatus){
        this.csStatus=csStatus;
      },

      //返回图片页面的方法
      backPicture:function () {
        //返回页面时要把笔记保存起来
        var that = this;
        this.$axios({//发送post请求，向后端数据库存储线条数组
          //请求路径：参数img_id为图片id，该笔记所在的图片的id
          url:'http://192.168.130.153:8887/CanvaController/saveNote?img_id='+that.pictures[that.pic_index].pic_id,
          method: 'post',                           //请求方式
          data:{                                    //提交给服务器的数据
            note:that.lineArr                       //线条数组
          },
          headers:{                                 //请求头
            'Content-Type': 'application/json'     //请求头中的数据编码类型
          }
        }).then(function (response) {              //请求发送成功的回调函数
          //向父组件App传递数据  两个参数：1.图片页面的状态true（显示） 2.画笔页面的状态false（隐藏）
          that.$emit("receiveCanva",true,false);
        },function (err) {
          console.log(err);
        });
      },

      //切换下一张图片，同时保存笔迹
      addIndex:function(){
        var that = this;
          this.$axios({//发送post请求，向后端数据库存储线条数组
            //请求路径：参数img_id为图片id，该笔记所在的图片的id
            url:'http://192.168.130.153:8887/CanvaController/saveNote?img_id='+that.pictures[that.pic_index].pic_id,
            method: 'post',                           //请求方式
            data:{                                    //提交给服务器的数据
              note:that.lineArr                       //线条数组
            },
            headers:{                                 //请求头
              'Content-Type': 'application/json'     //请求头中的数据编码类型
            }
          }).then(function (response) {              //请求发送成功的回调函数
            that.pic_index++;                        //下标index增加，切换到下一张图片
            that.readPoint();                        //调用查询笔记的方法，查询下一张图片的笔记，并将之绘制出来
          },function (err) {
            console.log(err);
          });
      },

      //切换上一张图片，同时保存笔迹
      subIndex:function(){
        var that = this;
          this.$axios({//发送post请求，向后端数据库存储线条数组
            //请求路径：参数img_id为图片id，该笔记所在的图片的id
            url:'http://192.168.130.153:8887/CanvaController/saveNote?img_id='+that.pictures[that.pic_index].pic_id,
            method: 'post',                          //请求方式
            data:{                                   //提交给服务器的数据
              note:that.lineArr                      //线条数组
            },
            headers:{                                //请求头
              'Content-Type': 'application/json'     //请求头中的数据编码类型
            }
          }).then(function (response) {              //请求发送成功的回调函数
            that.pic_index--;                        //下标index减小，切换到上一张图片
            that.readPoint();                        //调用查询笔记的方法，查询上一张图片的笔记，并将之绘制出来
          },function (err) {
            console.log(err);
          });
      },

      //画笔方法：画线
      paintBrush1: function () {
        var that = this;
        //画线状态，true为可以画线，false为不可以，这用于判断鼠标是否按下，当鼠标按下时可以画线，否则不可以
        var paintStatus = false;
        //画线的起始坐标，即在画布中按下鼠标的坐标
        let beginPoint = {x:undefined,y:undefined};
        //画线的终止坐标，即在画布中移动鼠标后的坐标，注意：鼠标移动事件在移动的时候是不断发生的，所以要不断更新起始坐标和终止坐标
        let endPoint = {x:undefined,y:undefined};
        /*给画布绑定鼠标按下事件：当按下鼠标时触发，记录坐标开始画线，同时把坐标保存到pointArr数组中*/
        this.canvas.onmousedown = function (e) {
            //计算起始坐标点
            var canvasLeft = e.target.offsetLeft; //画布的左边界到父元素（屏幕边缘）的距离
            var canvasTop = e.target.offsetTop;   //画布的上边界到父元素（屏幕边缘）的距离
            let x = e.clientX - canvasLeft;       //e.clientX：点击的坐标到屏幕左边缘的距离，两者相减：该坐标在画布中的X坐标
            let y = e.clientY - canvasTop;        //e.clientY：点击的坐标到屏幕上边缘的距离，两者相减：该坐标在画布中的Y坐标
            that.pointArr.push({                  //保存画笔起始坐标点到数组
              status:true,                        //用于判断是画笔还是橡皮擦，true为画笔，false为橡皮擦
              line_color: that.paintBrushColor,   //画笔颜色（画出的这条线的颜色）
              line_width: that.paintBrushWidth,   //画笔粗细（画出的这条线的粗细）
              point_X: x,                         //该起始坐标点的X坐标
              point_Y: y                          //该起始坐标点的Y坐标
            });
            // console.log(that.beginPoint);
            beginPoint = {"x": x, "y": y};        //获取起始坐标，即鼠标按下的坐标
            paintStatus = true;                   //设置画线状态为true
        };
        /*给画布绑定鼠标移动事件：当移动鼠标时触发，记录坐标点，然后画线，同时把坐标保存到pointArr数组中*/
        this.canvas.onmousemove = function (e) {
          if (paintStatus){                             //判断画布处于可绘画状态并且画线状态为true（即鼠标处于按下状态）
            var canvasLeft = e.target.offsetLeft;       //画布的左边界到父元素（屏幕边缘）的距离
            var canvasTop = e.target.offsetTop;         //画布的上边界到父元素（屏幕边缘）的距离
            let x = e.clientX-canvasLeft;               //e.clientX：点击的坐标到屏幕左边缘的距离，两者相减：该坐标在画布中的X坐标
            let y = e.clientY-canvasTop;                //e.clientY：点击的坐标到屏幕上边缘的距离，两者相减：该坐标在画布中的Y坐标
            that.pointArr.push({point_X:x,point_Y:y});  //保存坐标点到数组
            // console.log(that.pointArr);
            endPoint={"x":x,"y":y};//终止坐标
            that.drawLine(beginPoint.x,beginPoint.y,endPoint.x,endPoint.y);//调用画线方法画线:根据起始和终止坐标画线
            beginPoint=endPoint;//起始坐标更新，为下一次触发鼠标移动事件（画线）做准备
          }
        };
        /*给画布绑定鼠标离开事件：当鼠标离开画布时，将不能在画线*/
        this.canvas.mouseleave = function () {
          paintStatus=false;
        };
        /*给画布绑定鼠标松开事件：当鼠标松开时，将不能在画线，同时把画的这条线的坐标数组pointArr保存到线条数组lineArr*/
        this.canvas.onmouseup = function () {
          paintStatus=false;                       //设置画线状态为false
          that.lineArr.push(that.pointArr);        //把画的这条线的坐标数组pointArr保存到线条数组lineArr（保存画的线条）
          that.pointArr=[];                        //把坐标数组置空，为画下条线做准备
        }
      },

      //画线/绘制线条
      drawLine:function(x1,y1,x2,y2){              //参数：起始和终止坐标
        this.ctx.beginPath();                      //开启路径
        this.ctx.lineWidth = this.paintBrushWidth; //设置线条宽度，就是画线的线条宽度
        this.ctx.strokeStyle=this.paintBrushColor; //设置线条颜色
        this.ctx.lineCap="round";                  //设置绘制圆形的结束线帽
        this.ctx.lineJoin="round";                 //设置当两条线条交汇时，创建圆形边角
        this.ctx.moveTo(x1,y1);                    //设置线条的起始坐标
        this.ctx.lineTo(x2,y2);                    //设置线条的终止坐标
        this.ctx.stroke();                         //划线：绘制这条线
        this.ctx.closePath();                      //关闭路径
      },

      /*清空：把画布上的笔记清空，同时把线条数组清空（清除所有线条）*/
      clearAll:function () {
        //清空线条数组（或者：this.lineArr=[];)
        this.lineArr.splice(0,this.lineArr.length);
        //清空画布上的笔记
        this.ctx.clearRect(0,0,this.canvas.width,this.canvas.height);
      },

    /* 橡皮擦：和画笔方法类似，也是根据坐标点，不过是将坐标点上的笔记清除*/
      clearLine:function (id) {
        // this.img_id=id;
        this.canvas = this.$refs.myCanvas;               //获取画布元素
        var that = this;
        let clearPoint = {x: undefined, y: undefined};   //保存清除的坐标点
        var clearStatus = false;                         //清除状态，类似于画笔状态，判断是否可清除
        /*在画布中按下鼠标，记录清除的起始坐标*/
        this.canvas.onmousedown = function (e) {
          clearStatus = true;                            //设置可清除
          var canvasLeft = e.target.offsetLeft;          //画布的左边界到父元素（屏幕边缘）的距离
          var canvasTop = e.target.offsetTop;            //画布的上边界到父元素（屏幕边缘）的距离
          let x = e.clientX - canvasLeft;                //e.clientX：点击的坐标到屏幕左边缘的距离，两者相减：该坐标在画布中的X坐标
          let y = e.clientY - canvasTop;                 //e.clientY：点击的坐标到屏幕上边缘的距离，两者相减：该坐标在画布中的Y坐标
          that.pointArr.push({                           //保存橡皮擦起始坐标点到数组
            status:false,                                //用于判断是画笔还是橡皮擦，true为画笔，false为橡皮擦
            line_width: that.paintBrushWidth,            //橡皮擦宽度
            point_X: x,                                  //该起始坐标点的X坐标
            point_Y: y                                   //该起始坐标点的X坐标
          });
          console.log(that.pointArr);
          clearPoint = {"x": x, "y": y};                 //保存坐标到清除坐标点
                                                         //清除该坐标点的笔记
          that.ctx.clearRect(clearPoint.x, clearPoint.y, that.paintBrushWidth, that.paintBrushWidth);
        };
        /*鼠标移动事件，判断当清除状态为true时清除笔记*/
        this.canvas.onmousemove = function (e) {
          if (clearStatus) {
            var canvasLeft = e.target.offsetLeft;        //计算清除坐标点
            var canvasTop = e.target.offsetTop;
            let x = e.clientX - canvasLeft;
            let y = e.clientY - canvasTop;
            that.pointArr.push({                         //保存清除坐标点到数组
              point_X: x,
              point_Y: y
            });
            clearPoint = {"x": x, "y": y};               //保存坐标到清除坐标点
                                                         //清除该坐标点的笔记
            that.ctx.clearRect(clearPoint.x, clearPoint.y, that.paintBrushWidth, that.paintBrushWidth);
          }
        };
        /*给画布绑定鼠标离开事件：当鼠标离开画布时，将不能在清除*/
        this.canvas.mouseleave = function () {
          clearStatus = false;
        };
        /*给画布绑定鼠标松开事件：当鼠标松开时，将不能在清除，同时把清除的这条线的坐标数组pointArr保存到线条数组lineArr*/
        this.canvas.onmouseup = function () {
          clearStatus = false;                           //设置清除状态为false
          that.lineArr.push(that.pointArr);              //把清除笔记的这条线的坐标数组pointArr保存到线条数组lineArr（保存清除的线条）
          that.pointArr=[];                              //把坐标数组置空，为下条线做准备
        }
      },

      //查询笔记：从数据库中查询出笔记的坐标点，然后调用绘图的方法将笔记绘制出来
      readPoint:function () {
        //先把画布上的笔记清除
        this.ctx.clearRect(0,0,this.canvas.width,this.canvas.height);
        //获取当前图片的id
        var id = this.pictures[this.pic_index].pic_id;
        //将this保存，在回调函数中this会改变，所以要先把this保存起来
        var that = this;
        //从数据库中查询笔记
        axios.get("http://192.168.130.153:8887/CanvaController/findNote?id="+id)
          .then(function (response) {
            if (response.data.length===0 || response.data===undefined) {
              //笔记为空时把lineArr数组赋值为空数组
              that.lineArr=[];
            }else {
              // console.log(response);
              //保存笔记
              that.lineArr = response.data.note;
              // console.log(that.lineArr);
              //调用绘图方法将笔记绘制出来
              that.drawPoint(that.lineArr);
            }
          },function (err) {
            console.log(err);
          });
      },

      //绘图方法：用于绘制之前保存的笔记
      drawPoint:function (lineArr) {                          //参数：线条数组：二维，数组中保存了画线和清除的线条，通过此方法将线条重绘出来
        for (let i=0;i<lineArr.length;i++){                   //循环对象是线条数组，每次循环都是一个线条
          if (lineArr[i][0].status) {                         //判断起始坐标点状态，true是画线，false是清除（该坐标点的状态就是该线条的状态）
            for (let j = 0; j < lineArr[i].length - 1; j++) { //循环对象是线条，每次循环都是一个坐标点（画线）
              this.ctx.beginPath();                           //开启路径
              this.ctx.lineWidth = lineArr[i][0].line_width;  //设置线条宽度，就是划线的线条宽度
              this.ctx.strokeStyle = lineArr[i][0].line_color;//设置线条颜色
              this.ctx.lineCap = "round";                     //设置绘制圆形的结束线帽
              this.ctx.lineJoin = "round";                    //设置当两条线条交汇时，创建圆形边角
              this.ctx.moveTo(lineArr[i][j].point_X, lineArr[i][j].point_Y);    //设置线条的起始坐标
              this.ctx.lineTo(lineArr[i][j + 1].point_X, lineArr[i][j + 1].point_Y);//设置线条的终止坐标
              this.ctx.stroke();                              //划线：绘制这条线
              this.ctx.closePath();                           //关闭路径
            }
          }else {                                             //判断起始坐标点状态，true是画线，false是清除（该坐标点的状态就是该线条的状态）
            for (let j = 0; j < lineArr[i].length - 1; j++) { //循环对象是线条，每次循环都是一个坐标点（清除）
                                                              //清除坐标点上的笔记
              this.ctx.clearRect(lineArr[i][j].point_X, lineArr[i][j].point_Y, lineArr[i][0].line_width, lineArr[i][0].line_width);
            }
          }
        }
      },

    },

  }
</script>

<style>
  #canvas{
    position: relative;
    top: -8px;
    left: -8px;
  }
  #backBtn{
    position: absolute;
    left: 8px;
    top: 8px;
  }
  #canva_picture{
    width: 600px;
    margin: 0 auto;
    text-align: center;
    background-color: white;
  }
  #one_picture{
    position: absolute;
    top: 70px;
    left: 297px;
  }
  #myCanvas{
    position: absolute;
    top: 70px;
    left: 297px;
    z-index: 1;
  }
  #left{
    position: absolute;
    font-size: 50px;
    left: 290px;
    top: 232px;
    z-index: 2;
  }
  #right{
    position: absolute;
    font-size: 50px;
    z-index: 2;
    left: 968px;
    top: 232px;
  }
  #operation{
    position: absolute;
    top: 500px;
    left: 514px;
  }
  #chCloor{
    height: 19px;
  }
  #isColorSelector{
    width: 50px;
    height: 23px;
    position: absolute;
    left: 210px;
    border-color: #EDEDED;
  }
  #slider{
    position: absolute;
    width: 262px;
  }

</style>
