<template>
  <div id="Selector">
    <!--颜色选择器-->
    <div id="colorSelector" @mousedown="isMove" @mousemove="csMove" @mouseup="noMove">
      <!--标题-->
      <span id="colorSelectorTitle">颜色</span>
      <!--退出按钮-->
      <span id="colorSelectorQuitBtn" @click="quitColorSelect">x</span>
      <!--颜色表格-->
      <table border="0" id="colorTalbe" cellspacing="5px">
        <tr v-for="(item,index) in colors">
          <td v-for="(color,index) in item" :bgcolor="color" @click="chooseColor(color)"></td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
  export default {
    name:'ColorSelector',
    data(){
      return{
        colors:[//颜色数组：颜色选择器上的颜色
          ["#FFFFFF","#DDDDDD","#AAAAAA","#888888","#666666","#444444","#000000",
          "#FFB7DD","#FF88C2","#FF44AA","#FF0088","#C10066","#A20055","#8C0044"],
          ["#FFCCCC","#FF8888","#FF3333","#FF0000","#CC0000","#AA0000","#880000",
          "#FFC8B4","#FFA488","#FF7744","#FF5511","#E63F00","#C63300","#A42D00"],
          ["#FFDDAA","#FFBB66","#FFAA33","#FF8800","#EE7700","#CC6600","#BB5500",
          "#FFEE99","#FFDD55","#FFCC22","#FFBB00","#DDAA00","#AA7700","#886600"],
          ["#FFFFBB","#FFFF77","#FFFF33","#FFFF00","#EEEE00","#BBBB00","#888800",
          "#EEFFBB","#DDFF77","#CCFF33","#BBFF00","#99DD00","#88AA00","#668800"],
          ["#CCFF99","#BBFF66","#99FF33","#77FF00","#66DD00","#55AA00","#227700",
          "#99FF99","#66FF66","#33FF33","#00FF00","#00DD00","#00AA00","#008800"],
          ["#BBFFEE","#77FFCC","#33FFAA","#00FF99","#00DD77","#00AA55","#008844",
          "#AAFFEE","#77FFEE","#33FFDD","#00FFCC","#00DDAA","#00AA88","#008866"],
          ["#99FFFF","#66FFFF","#33FFFF","#00FFFF","#00DDDD","#00AAAA","#008888",
          "#CCEEFF","#77DDFF","#33CCFF","#00BBFF","#009FCC","#0088A8","#007799"],
          ["#CCDDFF","#99BBFF","#5599FF","#0066FF","#0044BB","#003C9D","#003377",
          "#CCCCFF","#9999FF","#5555FF","#0000FF","#0000CC","#0000AA","#000088"],
          ["#CCBBFF","#9F88FF","#7744FF","#5500FF","#4400CC","#2200AA","#220088",
          "#D1BBFF","#B088FF","#9955FF","#7700FF","#5500DD","#4400B3","#3A0088"],
          ["#E8CCFF","#D28EFF","#B94FFF","#9900FF","#7700BB","#66009D","#550088",
          "#F0BBFF","#E38EFF","#E93EFF","#CC00FF","#A500CC","#7A0099","#660077"],
        ],
        moveStatus:false,          //颜色选择器是否可以移动
        beginX:0,                  //起始点X坐标，即鼠标按下时的X坐标
        beginY:0,                  //起始点Y坐标，即鼠标按下时的Y坐标
        endX:0,                    //鼠标触发移动事件后的X坐标
        endY:0,                    //鼠标触发移动事件后的X坐标
      }
    },

    methods:{
      //选择颜色：传给父组件Canva（画笔颜色）
      chooseColor:function (color) {
        //向父组件传值  参数：color：选择的颜色   false：颜色选择器的状态（csStatus）
        this.$emit("receiveColor",color,false);
        //颜色选择器回到默认位置
        let colorSelector = document.getElementById("colorSelector");
        colorSelector.style.left = 221+"px";
        colorSelector.style.top = 100+"px";
      },

      //关闭颜色选择器
      quitColorSelect:function () {
        //向父组件Canva传值  传颜色选择器的状态（false） 关闭颜色选择器
        this.$emit("receiveStatus",false);
        //颜色选择器回到默认位置
        let colorSelector = document.getElementById("colorSelector");
        colorSelector.style.left = 221+"px";
        colorSelector.style.top = 100+"px";
      },

      //鼠标按下时调用：颜色选择器开始移动
      isMove:function (e) {
        //设置颜色选择器可移动
        this.moveStatus = true;
        //记录起始坐标
        this.beginX = e.clientX;
        this.beginY = e.clientY;
        // console.log(this.beginX+" "+ this.beginY);
      },

      //鼠标移动时调用：颜色选择器移动
      csMove:function (e) {
        if (this.moveStatus) {//判断颜色选择器是否可移动（即判断鼠标是否按下）
          //记录鼠标移动后的坐标点
          this.endX = e.clientX;
          this.endY = e.clientY;
          // console.log(this.endX+" "+ this.endY);
          //计算移动的距离
          let x = (this.endX-this.beginX);
          let y = (this.endY-this.beginY);
          // console.log(x+" "+y);
          /*移动颜色选择器*/
          //1.获取颜色选择器
          let colorSelector = document.getElementById("colorSelector");
          //2.获取颜色选择器中的css样式的值（left和top）
          /*获取css样式值分两种方式：IE和非IE （这里获取的时样式style）
          * IE：colorSelector.currentStyle （获取colorSelector的style）
          * 非IE：document.defaultView.getComputedStyle(colorSelector,null)(获取colorSelector的style）
          * 这里要进行一下判断，来解决浏览器兼容问题*/
          //获取style：进行判断，解决浏览器兼容问题
          if (colorSelector.currentStyle) {
            var style = ector.currentStyle;
          }else {
            var style = document.defaultView.getComputedStyle(colorSelector,null);
          }
          //获取css样式（style）中的left和top，转换为Number类型
          let left = parseInt(style.left);
          let top = parseInt(style.top);
          // console.log(left);
          // console.log(top);
          //3.更改颜色选择器中的样式（left和top):移动颜色选择器（设置left和top为原来的值加上移动的距离）
          colorSelector.style.left = left+x+"px";
          colorSelector.style.top = top+y+"px";
          // console.log(style.left);
          // console.log(style.top);
          //4.更新起始坐标点
          this.beginX = this.endX;
          this.beginY = this.endY;
        }
      },

      //鼠标抬起时调用：颜色选择器停止移动
      noMove:function () {
        //设置颜色选择器不以可移动
        this.moveStatus = false;
      }
    }
  }
</script>

<style>
  #Selector{
    position: relative;
    background-color: transparent;
    z-index: 3;
    width: 1263px;
    height: 700px;
  }
  #colorSelector{
    position: absolute;
    z-index: 3;
    border: 1px solid #BFBFBF;
    width: 408px;
    height: 319px;
    top: 100px;
    left: 221px;
    background-color: white;
  }
  #colorSelectorTitle{
    position: absolute;
    top: 2px;
    left: 4px;
  }
  #colorSelectorQuitBtn{
    position: absolute;
    display: block;
    left: 383px;
    top: 0px;
    font-size: 17px;
    text-align: center;
    width: 25px;
    height: 26px;
  }
  #colorSelectorQuitBtn:hover{
    background-color: red;
  }
  #colorTalbe{
    background-color: #EDEDED;
    position: absolute;
    top: 27px;
    left: 1px;
    z-index: 1;
  }
  td{
    width: 20px;
    height: 20px;
    border: 1px solid #AAAAAA;
  }
</style>
