<!-- <script setup>
import {reactive} from 'vue'
// 设置文件切片大小
const SIZE = 10 * 1024 * 1024; 
const data=reactive({
 file:null
})
const handleFileChange=(e)=>{
  // 获取文件数据
  const [file] = e.target.files;
  if (!file){
     console.log('没文件数据')
  } 
  data.file=file
  console.log(data.file)
}
const handleUpload=()=>{

}
const createFileChunk=(file, size = SIZE)=>{
  const fileChunkList = [];
  let cur = 0;
  while(cur<file.size){
    fileChunkList.push({file:file.slice(cur,cur+size)})
    cur+=size
  }
  return fileChunkList
}

</script>

<template>
<div>
    <input type="file" @change="handleFileChange" />
    <button @click="handleUpload">上传</button>
</div>
</template>

<style scoped>

</style> -->
