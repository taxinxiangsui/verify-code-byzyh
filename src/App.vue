<script setup lang="ts">
import VerifyCode from './components/verify-code/VerifyCode.vue';
import { ref } from 'vue'
const checkOk = ()=> {
  console.log(1);
  
}
const checkFail = ()=> {
  console.log(2);
  
}
const arrayList = ref([
  'https://t7.baidu.com/it/u=1956604245,3662848045&fm=193&f=GIF',
  'https://t7.baidu.com/it/u=2529476510,3041785782&fm=193&f=GIF'
])
</script>

<template>
  <VerifyCode @ok="checkOk" @fail="checkFail" :verifyImgList='arrayList'/>
</template>

<style scoped>

</style>
