<template>
  <div class="hello">
    <div style="position:absolute;z-index:1"><img src="../assets/laji.jpg" alt=""></div>
    <div class="box">
      <div class="chessborder" v-for="item in chessBorders" :key="item.id" @click="moveTo(item.X,item.Y)">
        <div class="chessman" v-if="item.name" @click="selectItem(item.X,item.Y,item.name,item.color)" :style={color:item.color,borderColor:item.BDcolor}>{{item.name}}</div>
      </div>
    </div>
    <div style="position:relative;left:600px;top:280px">
      <div>
        <el-button @click="PullBack" v-show="!gameOver" v-bind:disabled="moveHistory.length==0">悔棋</el-button>
      </div>
      <div>
        <el-button @click="rePlay" style="margin:10px 0">重来</el-button>
      </div>
      <span style="color:red" v-if="runComp">红方走</span>
      <span style="color:#000" v-if="!runComp">黑方走</span>
    </div>
  </div>
</template>

<script>
export default {
  data(){
    return{
      chessBorders:[],    //所有格子
      runComp:true,     //行动方,为true时红方走
      runItem:{},     //选中将要移动的单位
      running:false,      //为true时表示已选中要移动的单位,点击规则范围内的格子进行移动
      couldMove:[],     //选中要移动的单位后计算的可移动的格子
      lastLostItem: {},     //被吃掉的棋子
      moveHistory: [],     //移动记录
      gameOver: false
    }
  },
  methods:{
    /*
      棋子id从1开始，
      在使用setItem方法的时候是去改变chessBorder[]里的值，
      所以id需要-1，
      理想状态应该在setItem方法里id-1再赋值，
      初期没有想到
    */
    moveTo(x,y){   //移动到指定位置
      let id = (y-1)*9+x
      if(this.runItem.id!=id&&this.running){
        if(this.TestPlace(id)){
          if(this.chessBorders[id-1].name){
            this.lastLostItem = {id:id,name:this.chessBorders[id-1].name}
            this.moveHistory.push({move:this.runItem,lost:this.lastLostItem})
            if(this.chessBorders[id-1].name == '帅'){
              this.gameOver = true
            }
            if(this.chessBorders[id-1].name == '将'){
              this.gameOver = true
            }
          }else{
            this.moveHistory.push({move:this.runItem,to:id})
          }
          this.setItem(this.runItem.id-1)
          this.setItem(id-1,this.runItem.name,this.runItem.color)
          this.running = false
          this.runComp = !this.runComp
          this.couldMove = []
          if(this.gameOver){
            let text = this.runItem.color == "#000" ? '黑方胜':'红方胜'
            setTimeout(() => {
              alert(text)
            }, 100);
          }
        }else{
          console.log('移动的位置不合法')
        }
      }
    },
    TestPlace (sid) { // 判断移动的位置是否符合规则
      let n = false
      this.couldMove.forEach(item => {
        if(item.id == sid){
          n = true
        }
      })
      return n
    },
    getCouldMove(name,id,x,y){    //计算可移动到的位置
      this.couldMove = [];
      if(name == "車" || name == "炮"){
        let C = (name == "車")? 1:2
        this.moveTypeCP(x,y,C)
      }
      if(name == "馬"){
        this.moveTypeM(x,y,id)
      }
      if(name == "相" || name == "象"){
        this.moveTypeX(x,y,id)
      }
      if(name == "士" || name == "仕"){
        this.moveTypeS(x,y,id)
      }
      if(name == '将' || name == '帅'){
        this.moveTypeJ(x,y,id)
      }
      if(name == '兵' || name == '卒'){
        let B = (name == "兵")? 1:2
        this.moveTypeB(x,y,id,B)
      }
      // console.log("couluMove: ",this.couldMove)
    },
    moveTypeCP(x,y,C){    //‘車’(C=1)和‘炮’(C=2)的可移动位置,
      let addID = 0
      //  从选中位置向左遍历
      for(let xaxis = x-1;xaxis>0;xaxis--){
        addID = (y-1)*9+xaxis
        if(this.chessBorders[addID-1].name){
          /*
            '車',如果遍历到第一个非空位置的棋子颜色和选中棋子颜色不同,
            将该位置加入可移动列表并跳出for循环
           *
            '炮',遍历到非空位置后,继续遍历其后面的格子,遇到第一个非空位置
            的棋子颜色与选中棋子颜色不同,该位置加入可移动列表跳出for循环
          */
          if(C == 1&&this.chessBorders[addID-1].color!=this.runItem.color){
            this.couldMove.push({id:addID,X:xaxis,Y:y})
            break;
          }else if(C == 2){
            for(let extraX = xaxis-1;extraX>0;extraX--){
              let extraID = (y-1)*9+extraX
              if(this.chessBorders[extraID-1].name !== undefined){
                if(this.chessBorders[extraID-1].color!=this.runItem.color){
                  this.couldMove.push({id:extraID,X:extraX,Y:y})
                }
                break;
              }
            }
          }
          break;
        }else{
          this.couldMove.push({id:addID,X:xaxis,Y:y})
        }
      }
      //  从选中位置向右遍历
      for(let xaxis = x+1;xaxis<=9;xaxis++){
        addID = (y-1)*9+xaxis
        if(this.chessBorders[addID-1].name){
          if(C == 1&&this.chessBorders[addID-1].color!=this.runItem.color){
            this.couldMove.push({id:addID,X:xaxis,Y:y})
            break;
          }else if(C == 2){
            for(let extraX = xaxis+1;extraX<=9;extraX++){
              let extraID = (y-1)*9+extraX
              if(this.chessBorders[extraID-1].name !== undefined){
                if(this.chessBorders[extraID-1].color!=this.runItem.color){
                  this.couldMove.push({id:extraID,X:extraX,Y:y})
                }
                break;
              }
            }
          }
          break;
        }else{
          this.couldMove.push({id:addID,X:xaxis,Y:y})
        }
      }
      //  向上遍历
      for(let yaxis = y-1;yaxis>0;yaxis--){
        addID = (yaxis-1)*9+x
        if(this.chessBorders[addID-1].name){
          if(C == 1&&this.chessBorders[addID-1].color!=this.runItem.color){
            this.couldMove.push({id:addID,X:x,Y:yaxis})
            break;
          }else if(C == 2){
            for(let extraY = yaxis-1;extraY>0;extraY--){
              let extraID = (extraY-1)*9+x
              if(this.chessBorders[extraID-1].name){
                if(this.chessBorders[extraID-1].color!=this.runItem.color){
                  this.couldMove.push({id:extraID,X:x,Y:extraY})
                }
                break;
              }
            }
          }
          break;
        }else{
          this.couldMove.push({id:addID,X:x,Y:yaxis})
        }
      }
      //  向下遍历
      for(let yaxis = y+1;yaxis<=10;yaxis++){
        addID = (yaxis-1)*9+x
        if(this.chessBorders[addID-1].name){
          if(C == 1&&this.chessBorders[addID-1].color!=this.runItem.color){
            this.couldMove.push({id:addID,X:x,Y:yaxis})
            break;
          }else if(C == 2){
            for(let extraY = yaxis+1;extraY<=10;extraY++){
              let extraID = (extraY-1)*9+x
              if(this.chessBorders[extraID-1].name !== undefined){
                if(this.chessBorders[extraID-1].color!=this.runItem.color){
                  this.couldMove.push({id:extraID,X:x,Y:extraY})
                }
                break;
              }
            }
          }
          break;
        }else{
          this.couldMove.push({id:addID,X:x,Y:yaxis})
        }
      }
    },
    moveTypeM(x,y,id){    //'馬'可移动位置
      // console.log("左一",this.chessBorders[id-2])
      // console.log("右一",this.chessBorders[id])
      // console.log("上一",this.chessBorders[id-10])
      // console.log("下一",this.chessBorders[id+8])
      //  向左遍历
      if(x>2&&!this.chessBorders[id-2].name){
        if(y>1){
          if(!this.chessBorders[id-12].name||this.chessBorders[id-12].color!=this.runItem.color){
            this.couldMove.push({id:id-11,X:x-2,Y:y-1})
          }
        }
        if(y<10){
          if(!this.chessBorders[id+6].name||this.chessBorders[id+6].color!=this.runItem.color){
            this.couldMove.push({id:id+7,X:x-2,Y:y+1})
          }
        }
      }
      //  向右遍历
      if(x<8&&!this.chessBorders[id].name){
        if(y>1){
          if(!this.chessBorders[id-8].name||this.chessBorders[id-8].color!=this.runItem.color){
            this.couldMove.push({id:id-7,X:x+2,Y:y-1})
          }
        }
        if(y<10){
          if(!this.chessBorders[id+10].name||this.chessBorders[id+10].color!=this.runItem.color){
            this.couldMove.push({id:id+11,X:x+2,Y:y+1})
          }
        }
      }
      //  向上遍历
      if(y>2&&!this.chessBorders[id-10].name){
        if(x>1){
          if(!this.chessBorders[id-20].name||this.chessBorders[id-20].color!=this.runItem.color){
            this.couldMove.push({id:id-19,X:x-1,Y:y-2})
          }
        }
        if(x<9){
          if(!this.chessBorders[id-18].name||this.chessBorders[id-18].color!=this.runItem.color){
            this.couldMove.push({id:id-17,X:x+1,Y:y-2})
          }
        }
      }
      //  向下遍历
      if(y<9&&!this.chessBorders[id+8].name){
        if(x>1){
          if(!this.chessBorders[id+16].name||this.chessBorders[id+16].color!=this.runItem.color){
            this.couldMove.push({id:id+17,X:x-1,Y:y+2})
          }
        }
        if(x<9){
          if(!this.chessBorders[id+18].name||this.chessBorders[id+18].color!=this.runItem.color){
            this.couldMove.push({id:id+19,X:x+1,Y:y+2})
          }
        }
      }
    },
    moveTypeX(x,y,id){    //'相'可移动位置
      if(x==5){
        //左下位置
        if(!this.chessBorders[id+7].name&&(!this.chessBorders[id+15].name||this.chessBorders[id+15].color!=this.runItem.color)){
          this.couldMove.push({id:id+16,X:x-2,Y:y+2})
        }
        //右下位置
        if(!this.chessBorders[id+9].name&&(!this.chessBorders[id+19].name||this.chessBorders[id+19].color!=this.runItem.color)){
          this.couldMove.push({id:id+20,X:x+2,Y:y+2})
        }
        //左上位置
        if(!this.chessBorders[id-11].name&&(!this.chessBorders[id-21].name||this.chessBorders[id-21].color!=this.runItem.color)){
          this.couldMove.push({id:id-20,X:x-2,Y:y-2})
        }
        //右上位置
        if(!this.chessBorders[id-9].name&&(!this.chessBorders[id-17].name||this.chessBorders[id-17].color!=this.runItem.color)){
          this.couldMove.push({id:id-16,X:x+2,Y:y-2})
        }
      }
      if(y==1||y==6){
        if(!this.chessBorders[id+7].name&&(!this.chessBorders[id+15].name||this.chessBorders[id+15].color!=this.runItem.color)){
          this.couldMove.push({id:id+16,X:x-2,Y:y+2})
        }
        if(!this.chessBorders[id+9].name&&(!this.chessBorders[id+19].name||this.chessBorders[id+19].color!=this.runItem.color)){
          this.couldMove.push({id:id+20,X:x+2,Y:y+2})
        }
      }
      if(y==5||y==10){
        if(!this.chessBorders[id-11].name&&(!this.chessBorders[id-21].name||this.chessBorders[id-21].color!=this.runItem.color)){
          this.couldMove.push({id:id-20,X:x-2,Y:y-2})
        }
        if(!this.chessBorders[id-9].name&&(!this.chessBorders[id-17].name||this.chessBorders[id-17].color!=this.runItem.color)){
          this.couldMove.push({id:id-16,X:x+2,Y:y-2})
        }
      }
      if(x==1){
        if(!this.chessBorders[id-9].name&&(!this.chessBorders[id-17].name||this.chessBorders[id-17].color!=this.runItem.color)){
          this.couldMove.push({id:id-16,X:x+2,Y:y-2})
        }
        if(!this.chessBorders[id+9].name&&(!this.chessBorders[id+19].name||this.chessBorders[id+19].color!=this.runItem.color)){
          this.couldMove.push({id:id+20,X:x+2,Y:y+2})
        }
      }
      if(x==9){
        if(!this.chessBorders[id-11].name&&(!this.chessBorders[id-21].name||this.chessBorders[id-21].color!=this.runItem.color)){
          this.couldMove.push({id:id-20,X:x-2,Y:y-2})
        }
        if(!this.chessBorders[id+7].name&&(!this.chessBorders[id+15].name||this.chessBorders[id+15].color!=this.runItem.color)){
          this.couldMove.push({id:id+16,X:x-2,Y:y+2})
        }
      }
    },
    moveTypeS(x,y,id){    //'士'可移动位置
      if(id==4||id==6||id==22||id==24){   //红色方边角位置
        if(!this.chessBorders[13].name||this.chessBorders[13].color!=this.runItem.color){
          this.couldMove.push({id:14,X:5,Y:2})
        }
      }
      if(id==85||id==87||id==67||id==69){   //黑色方边角位置
        if(!this.chessBorders[76].name||this.chessBorders[76].color!=this.runItem.color){
          this.couldMove.push({id:77,X:5,Y:9})
        }
      }
      if(id==14||id==77){   //中间位置
        if(!this.chessBorders[id-11].name||this.chessBorders[id-11].color!=this.runItem.color){
          this.couldMove.push({id:id-10,X:x-1,Y:y-1})
        }
        if(!this.chessBorders[id-9].name||this.chessBorders[id-9].color!=this.runItem.color){
          this.couldMove.push({id:id-8,X:x+1,Y:y-1})
        }
        if(!this.chessBorders[id+7].name||this.chessBorders[id+7].color!=this.runItem.color){
          this.couldMove.push({id:id+8,X:x-1,Y:y+1})
        }
        if(!this.chessBorders[id+9].name||this.chessBorders[id+9].color!=this.runItem.color){
          this.couldMove.push({id:id+10,X:x+1,Y:y+1})
        }
      }
    },
    moveTypeJ(x,y,id){    //'将'可移动位置
      //向左检测
      if(x-1>=4&&(!this.chessBorders[id-2].name||this.chessBorders[id-2].color!=this.runItem.color)){
        this.couldMove.push({id:id-1,X:x-1,Y:y})
      }
      //向右检测
      if(x+1<=6&&(!this.chessBorders[id].name||this.chessBorders[id].color!=this.runItem.color)){
        this.couldMove.push({id:id+1,X:x+1,Y:y})
      }
      //向上检测
      if((y==2||y==3||y==9||y==10)&&(!this.chessBorders[id-10].name||this.chessBorders[id-10].color!=this.runItem.color)){
        this.couldMove.push({id:id-9,X:x,Y:y-1})
      }
      //向下检测
      if((y==1||y==2||y==8||y==9)&&(!this.chessBorders[id+8].name||this.chessBorders[id+8].color!=this.runItem.color)){
        this.couldMove.push({id:id+9,X:x,Y:y+1})
      }
      //特殊规则
      if(y<4){
        let i = id+9,
            newY = y+1
        for(i;i<90;){
          if(this.chessBorders[i-1].name){
            if(this.chessBorders[i-1].name == '将'){
              this.couldMove.push({id:i,X:x,Y:newY})
            }
            break
          }
          i += 9
          newY++
        }
      }
      if(y>7){
        let i = id-9,
            newY = y-1
        for(i;i>1;){
          if(this.chessBorders[i-1].name){
            if(this.chessBorders[i-1].name == '帅'){
              this.couldMove.push({id:i,X:x,Y:newY})
            }
            break
          }
          i -= 9
          newY--
        }
      }
    },
    moveTypeB(x,y,id,B){  //'兵'可移动位置,红色方B=1,黑色方B=2
      if(B==1){
        //过河前
        if(y<6&&(!this.chessBorders[id+8].name||this.chessBorders[id+8].color!=this.runItem.color)){
          this.couldMove.push({id:id+9,X:x,Y:y+1})
        }
        //过河后
        if(y>=6){
          //向左检测
          if(x>1&&(!this.chessBorders[id-2].name||this.chessBorders[id-2].color!=this.runItem.color)){
            this.couldMove.push({id:id-1,X:x-1,Y:y})
          }
          //向右检测
          if(x<9&&(!this.chessBorders[id].name||this.chessBorders[id].color!=this.runItem.color)){
            this.couldMove.push({id:id+1,X:x+1,Y:y})
          }
          //向下检测
          if(y<10&&(!this.chessBorders[id+8].name||this.chessBorders[id+8].color!=this.runItem.color)){
            this.couldMove.push({id:id+9,X:x,Y:y+1})
          }
        }
      }
      if(B==2){
        //过河前
        if(y>=6&&(!this.chessBorders[id-10].name||this.chessBorders[id-10].color!=this.runItem.color)){
          this.couldMove.push({id:id-9,X:x,Y:y-1})
        }
        //过河后
        if(y<6){
          //向左检测
          if(x>1&&(!this.chessBorders[id-2].name||this.chessBorders[id-2].color!=this.runItem.color)){
            this.couldMove.push({id:id-1,X:x-1,Y:y})
          }
          //向右检测
          if(x<9&&(!this.chessBorders[id].name||this.chessBorders[id].color!=this.runItem.color)){
            this.couldMove.push({id:id+1,X:x+1,Y:y})
          }
          //向上检测
          if(y>1&&(!this.chessBorders[id-10].name||this.chessBorders[id-10].color!=this.runItem.color)){
            this.couldMove.push({id:id-9,X:x,Y:y-1})
          }
        }
      }
    },
    setItem(id,name,color){   //设置格子内的棋子
      let c;
      if(color == 1||color == "#ff0000"){
        c = "#ff0000"
      }else{
        c = "#000"
      }
      this.$set(this.chessBorders[id],"name",name)
      this.$set(this.chessBorders[id],"color",c)
    },
    selectItem(x,y,name,color){    //选择棋子
      let id = (y-1)*9+x
      // console.log(id,x,y,name,color)
      let trueSelect = false
      if(this.runComp&&color == "#ff0000"){
        trueSelect = true
      }
      if(!this.runComp&&color == "#000"){
        trueSelect = true
      }
      if(trueSelect&&!this.gameOver){
        this.$set(this.chessBorders[id-1],"BDcolor",color)
        this.runItem = {
          name:name,
          color:color,
          id:id,
          X:x,
          Y:y
        }
        this.getCouldMove(name,id,x,y)
        this.running = true;
      }
    },
    PullBack(){     //悔棋
      if(this.moveHistory.length){
        let backItem = this.moveHistory.pop()
        let C = backItem.move.color == "#000"? 1:"#000"
        if(backItem.to){
          this.setItem(backItem.to-1)
        }else{
          this.setItem(backItem.lost.id-1,backItem.lost.name,C)
        }
        this.setItem(backItem.move.id-1,backItem.move.name,backItem.move.color)
        this.runComp = !this.runComp
      }
    },
    init(){   //初始化
      this.chessBorders = []
      for(let y=1;y<=10;y++){
        for(let x=1;x<=9;x++){
          this.chessBorders.push({X:x,Y:y})
        }
      }
      this.gameOver = false
      this.setItem(0,"車",1);
      this.setItem(1,"馬",1);
      this.setItem(2,"相",1);
      this.setItem(3,"仕",1);
      this.setItem(4,"帅",1);
      this.setItem(5,"仕",1);
      this.setItem(6,"相",1);
      this.setItem(7,"馬",1);
      this.setItem(8,"車",1);
      this.setItem(19,"炮",1);
      this.setItem(25,"炮",1);
      this.setItem(27,"兵",1);
      this.setItem(29,"兵",1);
      this.setItem(31,"兵",1);
      this.setItem(33,"兵",1);
      this.setItem(35,"兵",1);
      this.setItem(54,"卒");
      this.setItem(56,"卒");
      this.setItem(58,"卒");
      this.setItem(60,"卒");
      this.setItem(62,"卒");
      this.setItem(64,"炮");
      this.setItem(70,"炮");
      this.setItem(81,"車");
      this.setItem(82,"馬");
      this.setItem(83,"象");
      this.setItem(84,"士");
      this.setItem(85,"将");
      this.setItem(86,"士");
      this.setItem(87,"象");
      this.setItem(88,"馬");
      this.setItem(89,"車");
      this.moveHistory = []
      this.runComp = true
    },
    rePlay(){
      this.$confirm ('放弃本局重新开始游戏?', '重来', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.init()
      })
    }
  },
  mounted(){
    this.init()
  },
  watch:{
    runItem:function(newItem,oldItem){    //重新选择棋子时,去掉上一个选中的边框颜色
      if(oldItem.id&&oldItem.id!=newItem.id){
        let c = '#aaaaaa'
        let id = oldItem.id
        this.$set(this.chessBorders[id-1],"BDcolor",c)
      }
    }
  }
}
</script>

<style scoped>
.box{
  width:540px;
  height:600px;
  border:1px solid #cccccc;
  position:absolute;
  z-index: 2;
}
.chessborder{
  width: 60px;
  height: 60px;
  float: left;
}
.chooseman{
  background-color: #333;
  border-color: #aaaaaa
}
.chessman{
  width:50px;
  height:50px;
  line-height: 50px;
  text-align: center;
  font-size: 35px;
  font-weight: bolder;
  font-family: "KaiTi";
  background-color: #eeeeee;
  border:3px solid #bbbbbb;
  position: relative;
  top:2px;
  left:2px;
  border-radius:50%;
}
</style>
