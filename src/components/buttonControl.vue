<template>

  <el-space :size="size" :spacer="spacer">
    <el-button round icon="el-icon-arrow-right" v-on:click="$parent.$parent.$parent.$parent.nextInstruct()">单步执行</el-button>
    <el-button round icon="el-icon-d-arrow-right" v-on:click="$parent.$parent.$parent.$parent.keepInstruct()">连续执行</el-button>
    <el-button round icon="el-icon-refresh-right" v-on:click="$parent.$parent.$parent.$parent.restart()">重置</el-button>
  </el-space>
</template>

<script>
import { h} from 'vue'
import { ElDivider } from 'element-plus'
export default {
  name: "buttonControl",
  methods:{
  },
  data(){
    return{
      size: 10,
      spacer: h(ElDivider, { direction: 'vertical' }),
    }
  }
}



</script>

<style scoped>

</style>