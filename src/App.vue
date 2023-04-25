<template>
  <div class="webcam">
    <video controls autoplay playsinline src="" ref="video"></video>
    <canvas width="400" height="400" ref="canvas"></canvas>
  </div>
  <button @click="openWebcam">开启摄像头</button>
</template>

<script lang='ts' setup>
import { ref, reactive, toRefs, onBeforeMount } from 'vue';
import * as faceapi from 'face-api.js'


const video = ref<HTMLVideoElement>()
const canvas = ref<HTMLCanvasElement>()

onBeforeMount(() => {
  // faceapi.nets.ssdMobilenetv1
})

const models = './weights';
(async () => {
  await Promise.all([
    // faceapi.loadSsdMobilenetv1Model(models), // SSD Mobilenet V1 - ssd_mobilenetv1_model
    // faceapi.loadFaceLandmarkModel(models), // face_landmark_68_model
    // faceapi.loadFaceLandmarkTinyModel(models), // face_landmark_68_tiny_model
    faceapi.loadAgeGenderModel(models), //加载训练模型 Age and Gender Recognition Model
    faceapi.loadFaceDetectionModel(models),//加载训练模型 face_detector_model
    faceapi.loadFaceExpressionModel(models),//加载训练模型 Face Expression Recognition Model
    faceapi.loadTinyFaceDetectorModel(models),//加载训练模型 tiny_face_detector_model
    faceapi.loadFaceRecognitionModel(models),//加载训练模型 face_recognition_model
  ])
})()


const openWebcam = () => {
  
  navigator.mediaDevices.getUserMedia({ audio: true, video: true }).then(res => {
    console.log(res);

    video.value!.srcObject = res;
  })
  const context = canvas.value?.getContext('2d')
  setInterval(async () => {
    context?.drawImage(video.value as any, 0, 0, 400, 400);
    faceBox();
    // faceLandMask();
    // faceExpression();
    //获取分析人脸的数据
    // const detections = await faceapi.detectAllFaces(video.value as any).withFaceLandmarks();

    // const resizedDetections = faceapi.resizeResults(detections, { width: 400, height: 400 });
    // //将人脸边框绘制到canvas上
    // faceapi.draw.drawDetections(canvas.value as any, resizedDetections)
    // faceapi.draw.drawFaceLandmarks(canvas.value as any, resizedDetections)
  }, 100)
}
const faceBox = async () => {
  const detections = await faceapi.detectAllFaces(video.value as any)
  const resizedDetections = faceapi.resizeResults(detections, { width: 400, height: 400 });
  faceapi.draw.drawDetections(canvas.value as any, resizedDetections)
}
const faceLandMask = async () => {
  const detections = await faceapi.detectAllFaces(video.value as any).withFaceLandmarks();

const resizedDetections = faceapi.resizeResults(detections, { width: 400, height: 400 });
//将人脸边框绘制到canvas上
faceapi.draw.drawDetections(canvas.value as any, resizedDetections)
faceapi.draw.drawFaceLandmarks(canvas.value as any, resizedDetections)
}
const faceExpression =async () => {
  const detections = await faceapi.detectAllFaces(video.value as any).withFaceLandmarks().withFaceExpressions()
  const resizedDetections = faceapi.resizeResults(detections, { width: 400, height: 400 });
  faceapi.draw.drawDetections(canvas.value as any, resizedDetections)
  const minProbability = 0.05;
  faceapi.draw.drawFaceExpressions(canvas.value as any, resizedDetections, minProbability)
}
</script>

<style lang='less' scoped>
.webcam{
  width: 640px;
  height: 480px;
  position: relative;
  video {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  canvas {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 1;
  }
}
</style>
