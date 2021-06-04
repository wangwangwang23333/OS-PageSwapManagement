<template>
  <div id="app">

    <el-container>
      <el-aside style="background-color: rgb(238, 241, 246);box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1) ;" >
        <Information   :sum-instruction="sumInstruction" :replace-algorithm="replaceAlgorithm"
                       :execution-sequence="executionSequence" :instruction-each-page="instructionEachPage" :memory-page="memoryPage"
                       ref="information"/>
        <el-divider></el-divider>
        <p>缺页数</p>
        <h3>{{missingPage}}</h3>
        <p>缺页率</p>
        <h3>{{missingRate}}%</h3>
      </el-aside>

      <el-container style="height: 600px">
        <el-header style="text-align: right; font-size: 14px;">
          <span>操作系统课程设计 | 内存管理</span>
          <el-dropdown>
            <i class="el-icon-more" style="margin-left: 18px;margin-right: 20px"></i>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="instructInfo">说明</el-dropdown-item>
                <el-dropdown-item @click="exit">离开</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <i class="el-icon-s-custom" style="margin-left: 18px;margin-right: 2px"></i>
          <span>1851055 汪明杰</span>
        </el-header>
        <el-main>
          <el-space alignment="flex-start">
            <MemoryStation ref="memoryStation" align="center"/>
            <div>
              <h4>已执行指令</h4>
              <instruction-table ref="instructionTable" class="table_pos"/>
            </div>

          </el-space>

        </el-main>
        <!--<HelloWorld msg="Welcome to Your Vue.js App"/>-->
        <el-footer>
          <button-control/>
        </el-footer>
      </el-container>

    </el-container>

  </div>
</template>

<script>

import Information from "@/components/Information";
import MemoryStation from "@/components/memoryStation";
import InstructionTable from "@/components/instructionTable";
import ButtonControl from "@/components/buttonControl";
import gsap from 'gsap';
//import instructionTable from "@/components/instructionTable";
import { ElMessage } from 'element-plus'
export default {
  name: 'App',
  components: {
    ButtonControl,
    InstructionTable,
    MemoryStation,
    Information
  },
  created() {
    //给指令随机
    this.instruction =new Array(this.sumInstruction);
    for(let i=0;i<this.sumInstruction;++i){
      this.instruction[i]={
        //所在页码
        place:i,
        page:Math.floor(i/this.instructionEachPage),
        placeInPage:i%this.instructionEachPage,
        finished:false
      };
    }
    for(let i=0;i<this.memoryPage;++i){
      this.LRUTable.push(-1);
    }
  },
  watch:{
    missingPage(newValue){
      gsap.to(this.$data, { duration: 0.5, animatedPage: this.curInstruct==0 ?0:newValue/this.curInstruct*100 });
    }
  },
  computed:{
    missingRate:function (){
      return this.animatedPage.toFixed();
    }
  },
  methods:{
    //重置
    restart(){
      this.$confirm('确认重置指令状态?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        if(this.timerStart){
          clearInterval(this.timer);
          this.timerStart=false;
        }
        ElMessage.success({
          message: '已重置指令状态',
          type: 'success'
        });
        this.$refs.instructionTable.tableData.splice(0,this.$refs.instructionTable.tableData.length);
        this.$refs.memoryStation.blockEmpty();
        this.innerPage.splice(0,this.innerPage.length);
        this.FIFOPointer=0;
        this.skipState=0;
        this.lastPos=-1;
        for(let i=0;i<this.sumInstruction;++i){
          this.instruction[i].finished=false;
        }
        this.curInstruct=0;
        this.missingRate=0;
        this.missingPage=0;
        this.skipState=0;

      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        });
      });


    },
    findPos(){
      if(this.skipState==0){
        let i=-1;
        do{
          i=Math.floor(Math.random()*(this.sumInstruction)); //生成[0,320)之间的随机数
        }
        while(this.instruction[i].finished)
        this.skipState=1;
        this.lastPos=i;
        return i;
      }
      else if (this.skipState==1){
        if(this.lastPos+1==this.sumInstruction||this.instruction[this.lastPos+1].finished){
          //重新随机位置
          let i=-1;
          do{
            i=Math.floor(Math.random()*(this.sumInstruction)); //生成[0,320)之间的随机数
          }
          while(this.instruction[i].finished)
          this.skipState=1;
          this.lastPos=i;
          return i;
        }
        else {
          this.skipState=2;
          return this.lastPos+1;
        }
      }
      else if (this.skipState==2){
        //如果lastPos后面的指令都被执行过了的话
        let findIt=false;
        for(let i=this.lastPos;i<this.sumInstruction;++i){
          if(!this.instruction[i].finished){
            findIt=true;
            break;
          }
        }
        if(!findIt){
          //重新找位置
          let i=-1;
          do{
            i=Math.floor(Math.random()*(this.sumInstruction)); //生成[0,320)之间的随机数
          }
          while(this.instruction[i].finished)
          this.skipState=1;
          this.lastPos=i;
          return i;
        }
        //找后面的位置
        let i=-1;
        do{
          i=Math.floor(Math.random()*(this.sumInstruction-this.lastPos)+this.lastPos); //生成[lastPos,320)之间的随机数
        }
        while(this.instruction[i].finished)
        this.skipState=3;
        return i;
      }
      else if (this.skipState==3){
        //如果lastPos前面的指令都被执行过了的话
        let findIt=false;
        for(let i=0;i<this.lastPos;++i){
          if(!this.instruction[i].finished){
            findIt=true;
            break;
          }
        }
        if(!findIt){
          //重新找位置
          let i=-1;
          do{
            i=Math.floor(Math.random()*(this.sumInstruction)); //生成[0,320)之间的随机数
          }
          while(this.instruction[i].finished)
          this.skipState=1;
          this.lastPos=i;
          return i;
        }
        //找前面的位置
        let i=-1;
        do{
          i=Math.floor(Math.random()*(this.lastPos)); //生成[0,lastPos)之间的随机数
        }
        while(this.instruction[i].finished)
        this.skipState=0;
        return i;
      }

    },

    nextInstruct(){
      //已执行完全部指令
      if(this.curInstruct==this.sumInstruction){
        //提示信息
        ElMessage.success({
          message: '全部320条指令已执行完毕',
          type: 'success'
        });
        //关闭计时器
        if(this.timerStart){
          clearInterval(this.timer);
          this.timerStart=false;
        }
        return;
      }
      //读取下一条指令:根据读取状态
      let readState=this.$refs.information.sequenceValue;
      let nextInstruct;

      if(readState=="sequential") {
        for(let i=0;i<this.sumInstruction;++i){
          if(!this.instruction[i].finished){
            nextInstruct=this.instruction[i];
            this.instruction[i].finished=true;
            break;
          }
        }
      }
      else if (readState=="random"){
        let i=-1;
        do{
          i=Math.floor(Math.random()*(this.sumInstruction)); //生成[0,320)之间的随机数
        }
        while(this.instruction[i].finished)
        nextInstruct=this.instruction[i];
        this.instruction[i].finished=true;
      }
      else if (readState=="skip"){
        //随机选一个作为初始点
        let i=this.findPos();
        nextInstruct=this.instruction[i];
        this.instruction[i].finished=true;
      }

      nextInstruct['index']=this.curInstruct+1;

      //查看该指令所在页是否在内存中
      let exists=false;
      let i=0;
      for(i=0;i<this.innerPage.length;++i){
        if(this.innerPage[i]==nextInstruct['page']) {
          exists=true;
          break;
        }
      }


      if(exists){
        nextInstruct['loss']='否';
        nextInstruct['outpage']='-';
        nextInstruct['inpage']='-';
        this.$refs.memoryStation.move(i,nextInstruct['placeInPage']);
      }
      else {
        ++this.missingPage;
        nextInstruct['loss'] = '是';
        nextInstruct['inpage'] = nextInstruct['page'];
        //如果当前内存中的页数量小于最大页数
        if (this.innerPage.length < this.memoryPage) {
          //直接放入内存中
          this.innerPage.push(nextInstruct['page']);

          this.$refs.memoryStation.memoryAppear(this.innerPage.length - 1);
          this.$refs.memoryStation.changePage(this.innerPage.length-1,nextInstruct['page']);
          nextInstruct['outpage'] = '-';

          //变红
          this.$refs.memoryStation.move(this.innerPage.length-1,nextInstruct['placeInPage']);
        }
        //已经满了，需要调出页
        else {
          //根据information选择相关算法
          let outPage;
          if(this.$refs.information.algorithmValue=='FIFO'){
            console.log('通过FIFO算法进行调度');
            outPage=this.FIFO();
          }
          else if (this.$refs.information.algorithmValue=='LRU'){
            console.log('通过LRU算法进行调度');
            outPage=this.LRU();
          }

          this.$refs.memoryStation.changePage(outPage,nextInstruct['page']);
          nextInstruct['outpage'] = this.innerPage[outPage];
          this.innerPage[outPage]=nextInstruct['page'];

          //变红
          this.$refs.memoryStation.move(outPage,nextInstruct['placeInPage']);
        }



        //更新LRU表
        for(let i=0;i<this.LRUTable.length;++i){
          if(this.innerPage[i]==nextInstruct['page']){
            this.LRUTable[i]=this.curInstruct;
            break;
          }
        }

      }

      //加入新数据
      this.$refs.instructionTable.addData(nextInstruct);
      this.$refs.instructionTable.setCurrent();
      ++this.curInstruct;
    },

    keepInstruct(){
      if(this.curInstruct==this.sumInstruction){
        //提示信息
        ElMessage.success({
          message: '全部320条指令已执行完毕',
          type: 'success'
        });
        //关闭计时器
        if(this.timerStart){
          clearInterval(this.timer);
          this.timerStart=false;
        }
        return;
      }
      if(this.timerStart){
        clearInterval(this.timer);
        this.timerStart=false;
        ElMessage.success({
          message: '已关闭连续执行',
          type: 'success'
        });
      }
      else{
        ElMessage.success({
          message: '已开始连续执行',
          type: 'success'
        });
        this.timerStart=true;
        this.timer=setInterval(this.nextInstruct,10);
      }

    },

    FIFO(){
      //先进先出算法：调出最近未使用的页
      let outPage=this.FIFOPointer;
      ++this.FIFOPointer;
      if(this.FIFOPointer==this.innerPage.length){
        this.FIFOPointer=0;
      }
      return outPage;
    },

    LRU(){
      //最近未使用算法，记录每一块最近被使用的时间点
      let minData=this.sumInstruction+1;
      let minIndex=-1;
      for(let i=0;i<this.LRUTable.length;++i){
        if(this.LRUTable[i]<minData){
          minData=this.LRUTable[i];
          minIndex=i;
        }
      }
      return minIndex;
    },

    instructInfo(){

      this.$alert('本项目是软件学院2021年春季学期操作系统课程第二次【内存管理】项目<br>'+
          '在左侧设置好<b>页面置换算法</b>和<b>执行顺序</b>后，即可通过<b>单步执行</b>一次执行一条指令或者<b>连续执行</b>直到指令执行结束。' +
          '<br>同时，在左侧可以观察<b>缺页数</b>和<b>缺页数</b>', '项目说明', {
        confirmButtonText: '确定',
        dangerouslyUseHTMLString:true
      });

    },
    exit(){
      this.$notify({
        title: '再见',
        message: '下次再见',
        type: 'success'
      });
      ElMessage.success({
        message: '下次再见',
        type: 'success'
      });
    }
  },
  data:()=>({
    executionSequence:"SKIP",
    instructionEachPage:10,
    memoryPage:4,
    replaceAlgorithm:"FIFO",
    sumInstruction:320,
    instruction:null,
    curInstruct:0,
    innerPage:[], //内存中存在的页
    FIFOPointer:0, //最先进的指针
    LRUTable:[], //记录内存中每个页最近被访问的时间
    missingPage:0,
    animatedPage:0,
    timer:'',
    timerStart:false,
    skipState:0,
    lastPos:0
  })
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.table_pos{
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1) ;
}

.el-header {
  background-color: #B3C0D1;
  color: #333;
  line-height: 60px;
}
</style>
