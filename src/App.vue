<template>
  <div class="wrapper">
    <div class="command-area">
      <textarea name="" id="" cols="30" rows="10" v-model="commandText" />
      <button @click="execConvert">转换</button>
    </div>
    <div class="preview-area">
      <img src="/rotate.png" alt="">
      <img ref="output" alt="">
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import JSZip from "jszip";
// @ts-ignore
import * as Magick from "./assets/imagemagick/magickApi.js";


// calling ImageMagick with one source image, and command to rotate & resize image

const output = ref<HTMLImageElement>()
const commandText = ref('[ "convert", "srcFile.png", "-charcoal", "2", "out.icns" ]')
// 执行转换
async function execConvert() {
  // the image element where to load output images
  
  // fetch the input image and get its content bytes
  const fetchedSourceImage = await fetch("/rotate.png");
  const sourceBytes = new Uint8Array(await fetchedSourceImage.arrayBuffer());
  const inputFiles = [{ name: 'srcFile.png', content: sourceBytes }]

  let command: string[] = []
  try {
    command = JSON.parse(commandText.value)
  } catch (ex) {
    alert(ex)
    //throw ex
  }
  const processedFiles = await Magick.Call(inputFiles, command);
  // response can be multiple files (example split) here we know we just have one
  const firstOutputImage = processedFiles[0];

  output.value!.src = URL.createObjectURL(firstOutputImage["blob"]);

  // 将转换结果打包成zip下载
  const zip = new JSZip();
  zip.file("output.icns", firstOutputImage["blob"]);
  const content = await zip.generateAsync({ type: "blob" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(content);
  link.download = "output.zip";
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>

<style scoped></style>
